# listen

```go
type Stream struct {
	r *bufio.Reader
	net.Conn
}

func newStream(c net.Conn) *Stream {
	return &Stream{bufio.NewReader(c), c}
}

func (b *Stream) Peek(n int) ([]byte, error) {
	return b.r.Peek(n)
}

func (b *Stream) Read(p []byte) (int, error) {
	return b.r.Read(p)
}

func (b *Stream) Line() (string, error) {
	line, _, err := b.r.ReadLine()
	return string(line), err
}

func readBe[T any](r io.Reader, data T) (err error) {
	if err = binary.Read(r, binary.BigEndian, data); err != nil {
		log.Error().Err(err).Msg("readBe error")
	}
	return
}

func writeBe[T any](r io.Writer, data T) (err error) {
	if err = binary.Write(r, binary.BigEndian, data); err != nil {
		log.Error().Err(err).Msg("writeBe error")
	}
	return
}

func listen(option Option) error {
	var lc net.ListenConfig
	lc.SetMultipathTCP(true)
	lc.Control = func(_, _ string, c syscall.RawConn) error {
		var err error
		_ = c.Control(func(fd uintptr) {
			if option.nodelay {
				err = unix.SetsockoptInt(int(fd), unix.IPPROTO_TCP, unix.TCP_NODELAY, 1)
			}
			if option.fastOpen {
				err = unix.SetsockoptInt(int(fd), unix.IPPROTO_TCP, unix.TCP_FASTOPEN, 1)
			}
			if option.reuseAddr {
				err = unix.SetsockoptInt(int(fd), unix.SOL_SOCKET, unix.SO_REUSEADDR, 1)
				err = unix.SetsockoptInt(int(fd), unix.SOL_SOCKET, unix.SO_REUSEPORT, 1)
			}
		})
		return err
	}
	ln, err := lc.Listen(context.Background(), "tcp", option.addr)
	if err != nil {
		return err
	}

	log.Info().Str("addr", option.addr).Msg("server start")

	for {
		conn, err := ln.Accept()
		if err != nil {
			log.Error().Err(err).Msg("accept error")
			break
		}
		stream := newStream(conn)

		peek, _ := stream.Peek(1)
		log.Trace().Uint8("type", peek[0]).Str("from", conn.LocalAddr().String()).Msg("incomeing request")
		if peek[0] == 0x05 {
			go serveSocks(stream)
		} else {
			go serveHTTP(stream)
		}
	}
	return nil
}

```


# http

```go

var SUCCESS = []byte("HTTP/1.1 200 Connection Established\r\n\r\n")

const (
	CONNECT  = "CONNECT"
	LF       = "\r\n"
	PORT     = "80"
	HEADLEN  = 5
	HEADSIZE = 512
)

type Req struct {
	Method string
	Host   string
	Port   string
}

func newReq(line []string) *Req {
	req := new(Req)
	req.Port = PORT
	// first
	{
		// CONNECT www.google.com:443 HTTP/1.1\r\n
		// GET http://www.google.com/ HTTP/1.1\r\n
		tmps := strings.Split(line[0], " ")
		req.Method = tmps[0]
	}

	// second
	{

		// Host: www.google.com:443
		// Host: www.google.com
		tmps := strings.Split(line[1], ":")
		req.Host = strings.TrimSpace(tmps[1])
		if len(tmps) == 3 {
			req.Port = strings.TrimSpace(tmps[2])
		}
	}
	return req
}

func serveHTTP(stream *Stream) {
	headers := make([]string, 0, HEADLEN)
	for {
		line, err := stream.Line()
		if err != nil {
			log.Error().Err(err).Msg("read line error")
			return
		}

		if line == LF || len(line) == 0 {
			break
		}
		headers = append(headers, line)
	}

	req := newReq(headers)
	remoteAddr := net.JoinHostPort(req.Host, req.Port)

	remote, err := net.Dial("tcp", remoteAddr)
	if err != nil {
		log.Error().Err(err).Str("to", remoteAddr).Msg("connect error")
		return
	}

	if req.Method == CONNECT {
		_, _ = stream.Write(SUCCESS)
	} else {
		buf := bytes.Buffer{}
		buf.Grow(HEADSIZE)
		for _, line := range headers {
			if strings.HasPrefix(line, "Proxy") {
				continue
			}
			buf.WriteString(line)
			buf.WriteString(LF)
		}
		buf.WriteString(LF)
		_, _ = remote.Write(buf.Bytes())
	}
	relay(stream, remote)
}

```

# socks5

```go

var (
	SUC    = []byte{0x05, 0x00}
	SusRep = []byte{0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00}
	errRep = []byte{0x05, 0x04, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00}
)

func serveSocks(stream *Stream) {
	header := make([]byte, 2)
	_ = readBe(stream, &header)

	nmeth := header[1]

	methods := make([]byte, int(nmeth))
	_ = readBe(stream, &methods)

	_ = writeBe(stream, SUC)

	// read request

	request := make([]byte, 4)
	_ = readBe(stream, &request)

	adtp := request[3]

	var host string

	switch adtp {
	case 1:
		hostBuf := make([]byte, net.IPv4len)
		_ = readBe(stream, &hostBuf)
		host = net.IP(hostBuf).String()
	case 3:
		hostBuf := make([]byte, net.IPv6len)
		_ = readBe(stream, &hostBuf)
		host = net.IP(hostBuf).String()
	default:
		var length uint8
		_ = readBe(stream, &length)
		buf := make([]byte, length)
		_ = readBe(stream, &buf)
		host = string(buf)
	}

	portBuf := make([]byte, 2)
	_ = readBe(stream, &portBuf)
	port := binary.BigEndian.Uint16(portBuf)

	targetAddr := net.JoinHostPort(host, strconv.FormatInt(int64(port), 10))

	remote, err := net.Dial("tcp", targetAddr)
	if err != nil {
		log.Error().Err(err).Str("remote", targetAddr).Msg("connect error")
		_, _ = stream.Write(errRep)
		return
	}
	_, _ = stream.Write(SusRep)
	relay(stream, remote)
}

```

# relay

```go
type Resp struct {
	len  int64
	err  error
	from net.Addr
	to   net.Addr
}

func relay(from, to net.Conn) {
	defer func(from, to net.Conn) {
		_ = from.Close()
		_ = to.Close()
	}(from, to)

	channel := make(chan Resp, 2)

	go func(dst, src net.Conn) {
		size, err := io.Copy(dst, src)
		channel <- Resp{len: size, err: err, from: src.RemoteAddr(), to: dst.RemoteAddr()}
	}(from, to)

	go func(dst, src net.Conn) {
		size, err := io.Copy(dst, src)
		channel <- Resp{len: size, err: err, from: src.RemoteAddr(), to: dst.RemoteAddr()}
	}(to, from)

	for resp := range channel {
		if resp.err != nil && resp.err != io.EOF {
			log.Error().Err(resp.err).Str("from", resp.from.String()).Str("to", resp.to.String()).Msg("relay error")
		} else {
			log.Info().Str("from", resp.from.String()).Str("to", resp.to.String()).Int64("transfer", resp.len).Msg("relay success")
		}
	}
}

```
