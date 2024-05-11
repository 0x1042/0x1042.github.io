
# 时间基本使用 

## 获取指定时区的时间 

```c++
        absl::TimeZone local;
        absl::LoadTimeZone("Asia/Shanghai", &local);
        auto now_in_second = absl::ToCivilSecond(absl::Now(), local);
        std::clog << "now_in_second:" << now_in_second << '\n';
```


## 获取指定时区当前小时

```c++
        absl::TimeZone local;
        absl::LoadTimeZone("Asia/Shanghai", &local);
        auto now_in_hour = absl::ToCivilHour(absl::Now(), local);
        std::clog << "now_in_hour:" << now_in_hour.hour() << '\n';
```

## 获取日期 

```c++
    absl::Duration one_day = absl::Hours(24) * offset;
    absl::Time tt = absl::Now() + one_day;
    absl::TimeZone local;
    absl::LoadTimeZone("Asia/Shanghai", &local);
    auto today = absl::ToCivilDay(tt, local);
    return absl::FormatCivilTime(today);
```

## 解析字符串到时间 

```c++
auto str2time(const std::string & str, absl::Time & dst, absl::TimeZone tz) -> bool {
    std::string err;
    bool status = absl::ParseTime("%Y-%m-%d", str, tz, &dst, &err);
    return status;
}

auto parse_time(const std::string & str) -> absl::CivilDay {
    absl::CivilDay day;

    bool status = absl::ParseCivilTime(str, &day);
    std::clog << "status:" << status << '\n';
    return day;
}
```

## 时间计算

```c++
        const std::string & start_str = "2024-05-10";
        auto start = parse_time(start_str);

        const std::string & end_str = "2024-05-18";
        auto end = parse_time(end_str);

        std::clog << "(end - start) = " << end - start << '\n';
        ASSERT_EQ(end - start, 8);
```