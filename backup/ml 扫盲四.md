# origin post

> https://x.com/konradgajdus/status/1837196363735482396

- [origin post](#origin-post)
- [process data](#process-data)
  - [读取images](#读取images)
  - [读取labels](#读取labels)
- [定义网络](#定义网络)
  - [结构](#结构)
  - [初始化](#初始化)
- [前向传播](#前向传播)
- [反向传播](#反向传播)
- [训练](#训练)
  - [单个instance单次迭代](#单个instance单次迭代)
  - [batch训练](#batch训练)
- [预测](#预测)
- [验证](#验证)

# process data

> 数据下载地址 https://yann.lecun.com/exdb/mnist/

| 文件                    | 说明       |
| ----------------------- | ---------- |
| `train-images-idx3-ubyte` | 训练集图片 |
| `train-labels-idx1-ubyte` | 训练集标签 |
| `t10k-images-idx3-ubyte`  | 测试集图片 |
| `t10k-labels-idx1-ubyte`  | 测试集标签 |


## 读取images

> 数据格式 

```
[offset] [type]          [value]          [description]
0000     32 bit integer  0x00000803(2051) magic number
0004     32 bit integer  60000            number of images
0008     32 bit integer  28               number of rows
0012     32 bit integer  28               number of columns
0016     unsigned byte   ??               pixel
0017     unsigned byte   ??               pixel
........
xxxx     unsigned byte   ??               pixel
Pixels are organized row-wise. Pixel values are 0 to 255. 0 means background (white), 255 means foreground (black).
```

```c
void read_images(const char *fname, unsigned char **images, int *count) {
	FILE *file = fopen(fname, "rb");
	if (file == NULL) {
		printf("%s open %s failed.", __FUNCTION__, fname);
		exit(1);
	}

	int magic = 0;
	int rows, cols;
	// magic numx
	fread(&magic, sizeof(int), 1, file);
	// number of images
	fread(count, sizeof(int), 1, file);
	// 转成大端
	*count = __builtin_bswap32(*count);

	// number of rows
	fread(&rows, sizeof(int), 1, file);
	//	number of columns
	fread(&cols, sizeof(int), 1, file);

	rows = __builtin_bswap32(rows);
	cols = __builtin_bswap32(cols);

	printf("%s: %d/%d/%d\n", __FUNCTION__, *count, rows, cols);

	*images = malloc((*count) * IMAGE_SIZE * IMAGE_SIZE);
	fread(*images, sizeof(unsigned char), (*count) * IMAGE_SIZE * IMAGE_SIZE, file);
	fclose(file);
}
```

## 读取labels

> 数据格式

```
[offset] [type]          [value]          [description]
0000     32 bit integer  0x00000801(2049) magic number (MSB first)
0004     32 bit integer  60000            number of items
0008     unsigned byte   ??               label
0009     unsigned byte   ??               label
........
xxxx     unsigned byte   ??               label
The labels values are 0 to 9.
```

```c
void read_labels(const char *fname, unsigned char **labels, int *count) {
	FILE *file = fopen(fname, "rb");
	if (file == NULL) {
		exit(1);
	}
	printf("open file [%s] success.\n", fname);

	int magic = 0;
	fread(&magic, sizeof(int), 1, file);
	fread(count, sizeof(int), 1, file);
	*count = __builtin_bswap32(*count);

	printf("labels count. %d.\n", *count);

	*labels = malloc(*count);

	fread(*labels, sizeof(unsigned char), *count, file);
	fclose(file);
}
```

# 定义网络 

## 结构 

```c
typedef struct {
	float *weights; // 权重 
	float *biases;  // 偏差
	int input_size;
	int output_size;
} Layer;

typedef struct {
	Layer hidden;
	Layer output;
} Network;
```

## 初始化 

> 神经网络的训练过程本质是对权重参数的更新，那么这个权重的初始值是什么?
> 首先不能是0，因为 $y = wx +b $ 中，如果w = 0，那么所有神经元的输出是相同的 反向传播过程的梯度也是相同的 

**He initialization**

> 思想是将权重初始化为满足一个标准正态分布

```c
void init_layer(Layer *layer, int in_size, int out_size) {
	int n = in_size * out_size;
	float scale = sqrtf(2.0f / in_size);

	layer->input_size = in_size;
	layer->output_size = out_size;
	layer->weights = malloc(n * sizeof(float));
	layer->biases = calloc(out_size, sizeof(float));

	for (int i = 0; i < n; i++)
		layer->weights[i] = ((float)rand() / RAND_MAX - 0.5f) * 2 * scale;
}
```

# 前向传播

> 即给定输入 计算神经网络输出的过程 
> 步骤: 将输入数据移动到网络的每一层，应用线形变换和激活函数，产生输出

```c
void forward(Layer *layer, float *input, float *output) {
	for (int i = 0; i < layer->output_size; i++) {
		output[i] = layer->biases[i];
		for (int j = 0; j < layer->input_size; j++)
			output[i] += input[j] * layer->weights[j * layer->output_size + i];
	}
}

void relu(float *hidden, int size) {
	for (int i = 0; i < size; i++) {
		hidden[i] = hidden[i] > 0 ? hidden[i] : 0;
	}
}

void softmax(float *input, int size) {
	float max = input[0], sum = 0;
	for (int i = 1; i < size; i++) {
		if (input[i] > max) max = input[i];
	}
	for (int i = 0; i < size; i++) {
		input[i] = expf(input[i] - max);
		sum += input[i];
	}
	for (int i = 0; i < size; i++) {
		input[i] /= sum;
	}
}
```

# 反向传播 

> 根据梯度更新权重和偏差
> 步骤:

```c
void backward(Layer *layer, float *input, float *output_grad, float *input_grad, float lr) {
	for (int i = 0; i < layer->output_size; i++) {
		for (int j = 0; j < layer->input_size; j++) {
			int idx = j * layer->output_size + i;
			// 相对于权重的损失梯度等于相对于输出的损失梯度乘以输入值
			float grad = output_grad[i] * input[j];
			// 新的权重 = 旧的权重 减去 学习率乘以相对于权重的损失梯度
			layer->weights[idx] -= lr * grad;
			//	相对于输入 j 的损失梯度是所有输出的（关于每个输出 i 的损失梯度乘以将输入 j 连接到输出 i 的权重）的总和
			if (input_grad) {
				input_grad[j] += output_grad[i] * layer->weights[idx];
			}
		}

		// 新偏差等于旧偏差减去学习率乘以相对于偏差的损失梯度。
		layer->biases[i] -= lr * output_grad[i];
	}
}
```

# 训练 

## 单个instance单次迭代

```c
/**
 * @brief 对单个instance 的训练(单张图片)
 */
void train(Network *net, float *input, int label, float lr) {
	float hidden_output[HIDDEN_SIZE], final_output[OUTPUT_SIZE];
	float output_grad[OUTPUT_SIZE] = {0}, hidden_grad[HIDDEN_SIZE] = {0};

    // 前向传播，将输入的图片传入隐藏层
	forward(&net->hidden, input, hidden_output);
    relu(hidden_output, HIDDEN_SIZE);

    // 隐藏层到输出层 
	forward(&net->output, hidden_output, final_output);
	softmax(final_output, OUTPUT_SIZE);

	// 损失函数
    // 计算输出梯度
	for (int i = 0; i < OUTPUT_SIZE; i++)
		output_grad[i] = final_output[i] - (i == label);

    // 反向传播，输出层到隐藏层 
	backward(&net->output, hidden_output, output_grad, hidden_grad, lr);

	for (int i = 0; i < HIDDEN_SIZE; i++)
		hidden_grad[i] *= hidden_output[i] > 0 ? 1 : 0;// ReLU derivative

    // 反向传播，隐藏层到输入层 
	backward(&net->hidden, input, hidden_grad, NULL, lr);
}
```

## batch训练

```c
	for (int epoch = 0; epoch < EPOCHS; epoch++) {
		float total_loss = 0;
		for (int i = 0; i < train_size; i += BATCH_SIZE) {
			for (int j = 0; j < BATCH_SIZE && i + j < train_size; j++) {
				int idx = i + j;
				for (int k = 0; k < INPUT_SIZE; k++)
					img[k] = data.images[idx * INPUT_SIZE + k] / 255.0f;

				train(&net, img, data.labels[idx], learning_rate);

				float hidden_output[HIDDEN_SIZE], final_output[OUTPUT_SIZE];
				forward(&net.hidden, img, hidden_output);
				for (int k = 0; k < HIDDEN_SIZE; k++)
					hidden_output[k] = hidden_output[k] > 0 ? hidden_output[k] : 0;// ReLU
				forward(&net.output, hidden_output, final_output);
				softmax(final_output, OUTPUT_SIZE);

				total_loss += -logf(final_output[data.labels[idx]] + 1e-10f);
			}
		}
	}
```

# 预测

```c

int predict(Network *net, float *input) {
	float hidden_output[HIDDEN_SIZE], final_output[OUTPUT_SIZE];

    // 前向传播，将输入的图片传入隐藏层
	forward(&net->hidden, input, hidden_output);
    relu(hidden_output, HIDDEN_SIZE);

    // 隐藏层到输出层 
	forward(&net->output, hidden_output, final_output);
	softmax(final_output, OUTPUT_SIZE);

	int max_index = 0;
	for (int i = 1; i < OUTPUT_SIZE; i++)
		if (final_output[i] > final_output[max_index])
			max_index = i;

	return max_index;
}
```

# 验证

```c

typedef struct {
	unsigned char *images;
	unsigned char *labels;
	int image_count;
	int label_count;
} InputData;

// 读取测试数据
InputData testdata = {0};
read_mnist_images(TEST_IMG_PATH, &testdata.images, &testdata.nImages);
read_mnist_labels(TEST_LBL_PATH, &testdata.labels, &testdata.nImages);

{
	float tmp[INPUT_SIZE];
	int corrent = 0;
	int total = testdata.nImages;
	for (int i = 0; i < testdata.nImages; i++) {
		for (int k = 0; k < INPUT_SIZE; k++)
			tmp[k] = testdata.images[i * INPUT_SIZE + k] / 255.0f;
		int pre = predict(&net, tmp);
		int post = testdata.labels[i];
		corrent += (pre == post);
	}
	printf("rate %.4f\n", (float) corrent / (float) total);
}
```

<img width="754" alt="image" src="https://github.com/user-attachments/assets/34eb1ea4-d2b1-475a-896b-cf5b64c109bd">
