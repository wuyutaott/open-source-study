## 项目地址
https://github.com/GoldBaby5511/go-mango-core

### 项目结构
- amqp 消息队列
- api 网络消息由proto文件生成
- chanrpc 轻量级模块之间通讯的工具，没做修改
- conf 配置
    - apollo 网络配置
- database
    - mchelper memcache操作
    - mgohelper mongodb操作
    - redishelper redis操作
    - sqlhelper mysql操作

### 知识点列表
- amqp 高级消息队列协议
    - rabbitmq rabbitmq是AMQP协议的实现者 [参考](http://t.zoukankan.com/wutianqi-p-10043011.html)
- 自动化脚本
    - proto生生脚本`build.bat`
- docker
- apollo 以ActiveMQ原型为基础，是一个更快、更可靠、更易于维护的消息代理工具。
- colorprint 输出有颜色的字体
- 叮叮相关操作
- aes加密解密
- rsa加密相关

### 知识点 - 01
检查端口是否被占用
```go
func PortInUse(portNumber int) bool {
	p := strconv.Itoa(portNumber)
	addr := net.JoinHostPort("127.0.0.1", p)
	conn, err := net.DialTimeout("tcp", addr, 3*time.Second)
	if err != nil {
		return false
	}
	defer conn.Close()

	return true
}
```

### 知识点 - 02
获取当前程序执行的路径
```go
func GetCurrentPath() (string, error) {
	file, err := exec.LookPath(os.Args[0])
	if err != nil {
		return "", err
	}
	path, err := filepath.Abs(file)
	if err != nil {
		return "", err
	}
	i := strings.LastIndex(path, "/")
	if i < 0 {
		i = strings.LastIndex(path, "\\")
	}
	if i < 0 {
		return "", errors.New(`error: Can't find "/" or "\".`)
	}
	return string(path[0 : i+1]), nil
}
```

### 知识点 - 03
级联创建文件夹
```go
err = os.MkdirAll(pathName, os.ModePerm)
```

### 知识点 - 04
暴力停服
```go
os.Exit(1)
```

### 知识点 - 05
获取当前的内存
```go
func CurMemory() int64 {
	var rtm runtime.MemStats
	runtime.ReadMemStats(&rtm)
	return int64(rtm.Alloc / 1024)
}
```

### 知识点 - 06
获取UUID
```go
func GetUUID() string {
	return uuid.New().String()
}
```

### 知识点 - 07
获取指定长度的随机字符串
```go
func RandByte(length int) []byte {
	var chars = []byte{'.', '/', '?', '%', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '1', '2', '3', '4', '5', '6', '7', '8', '9', '0'}
	buffer := bytes.Buffer{}
	clength := len(chars)
	rand.Seed(time.Now().UnixNano()) // 重新播种，否则值不会变
	for i := 0; i < length; i++ {
		buffer.WriteByte(chars[rand.Intn(clength)])
	}
	return buffer.Bytes()
}
```

### 知识点 - 08
深拷贝
```go
func deepCopy(dst, src reflect.Value) {
	switch src.Kind() {
	case reflect.Interface:
		value := src.Elem()
		if !value.IsValid() {
			return
		}
		newValue := reflect.New(value.Type()).Elem()
		deepCopy(newValue, value)
		dst.Set(newValue)
	case reflect.Ptr:
		value := src.Elem()
		if !value.IsValid() {
			return
		}
		dst.Set(reflect.New(value.Type()))
		deepCopy(dst.Elem(), value)
	case reflect.Map:
		dst.Set(reflect.MakeMap(src.Type()))
		keys := src.MapKeys()
		for _, key := range keys {
			value := src.MapIndex(key)
			newValue := reflect.New(value.Type()).Elem()
			deepCopy(newValue, value)
			dst.SetMapIndex(key, newValue)
		}
	case reflect.Slice:
		dst.Set(reflect.MakeSlice(src.Type(), src.Len(), src.Cap()))
		for i := 0; i < src.Len(); i++ {
			deepCopy(dst.Index(i), src.Index(i))
		}
	case reflect.Struct:
		typeSrc := src.Type()
		for i := 0; i < src.NumField(); i++ {
			value := src.Field(i)
			tag := typeSrc.Field(i).Tag
			if value.CanSet() && tag.Get("deepcopy") != "-" {
				deepCopy(dst.Field(i), value)
			}
		}
	default:
		dst.Set(src)
	}
}

func DeepCopy(dst, src interface{}) {
	typeDst := reflect.TypeOf(dst)
	typeSrc := reflect.TypeOf(src)
	if typeDst != typeSrc {
		panic("DeepCopy: " + typeDst.String() + " != " + typeSrc.String())
	}
	if typeSrc.Kind() != reflect.Ptr {
		panic("DeepCopy: pass arguments by address")
	}

	valueDst := reflect.ValueOf(dst).Elem()
	valueSrc := reflect.ValueOf(src).Elem()
	if !valueDst.IsValid() || !valueSrc.IsValid() {
		panic("DeepCopy: invalid arguments")
	}

	deepCopy(valueDst, valueSrc)
}

func DeepClone(v interface{}) interface{} {
	dst := reflect.New(reflect.TypeOf(v)).Elem()
	deepCopy(dst, reflect.ValueOf(v))
	return dst.Interface()
}
```

### 知识点 - 09
扑克数据结构
```go
var cardData = []byte{
	0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, //方块 A - K
	0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 0x18, 0x19, 0x1A, 0x1B, 0x1C, 0x1D, //梅花 A - K
	0x21, 0x22, 0x23, 0x24, 0x25, 0x26, 0x27, 0x28, 0x29, 0x2A, 0x2B, 0x2C, 0x2D, //红桃 A - K
	0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x3A, 0x3B, 0x3C, 0x3D, //黑桃 A - K
	0x4E, 0x4F
}
```