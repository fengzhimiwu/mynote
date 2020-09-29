### Go-kit 和 gRPC 结合的原理
在前面的课时中，我们介绍过 Go-kit 是一套强大的微服务开发工具集，用于指导开发人员解决分布式系统开发过程中所遇到的问题，帮助开发人员更专注于业务开发。Go-kit 推荐使用 Transport、Endpoint 和 Service 这 3 层结构来组织项目。若是你对这三者的具体概念还不熟悉的话，可以去复习一下前边的课时。

由于本课时主要涉及 Transport 层和 Endpoint 层，所以这里我们就再重申一下这二者的概念。

- Transport 层，主要负责网络传输，例如处理HTTP、gRPC、Thrift等相关的逻辑。

- Endpoint 层，主要负责 request/response 格式的转换，以及公用拦截器相关的逻辑。作为 Go-kit 的核心，Endpoint 层采用类似洋葱的模型，提供了对日志、限流、熔断、链路追踪和服务监控等方面的扩展能力。


**Go-kit 和 gRPC 结合的关键在于需要将 gRPC 集成到 Go-kit 的 Transport 层**。Go-kit 的 Transport 层用于接收用户网络请求并将其转为 Endpoint 可以处理的对象，然后交由 Endpoint 层执行，最后再将处理结果转为响应对象返回给客户端。为了完成这项工作，Transport 层需要具备两个工具方法：

- 解码器，把用户的请求内容转换为请求对象；

- 编码器，把处理结果转换为响应对象。


gRPC 请求的处理过程如下图所示，服务端接收到一个客户端请求后，交由 grpc_transport.Handler 处理，它会调用 DecodeRequestFunc 进行解码，然后交给其 Endpoint 层转换为 Service 层能处理的对象，将返回值通过 EncodeResponseFunc 编码，最后返回给客户端。

![](../../../images/go/microservice/service-52.jpg)

接下来，我们就按照上述的流程，实现通过 Go-kit 进行 gRPC 调用。

### Go-kit 集成 gRPC 案例实战
本课时中，我们的案例使用的是上一课时案例中所定义的 proto 文件并生成相应的代码，这里不再赘述。下面我们主要分五个步骤来构建基于 gRPC 的 Go-kit 项目，分别为：定义并构建Service、Endpoint、Middleware、Transport 和组装服务端。

1. 定义 Service，提供业务实现
首先定义了 UserService 结构，它有一个名为 CheckPassword 的 grpc_transport.Handler 的方法。这个方法会调用 grpc_transport.Handler 的 ServeGRPC 方法来将请求交由 Go-kit 处理。

```
// 定义接口
type UserService interface {
    CheckPassword(ctx context.Context, username string, password string) (bool, error);
}
​
type UserServiceImpl struct {}
​
func (s UserServiceImpl) CheckPassword(ctx context.Context, username string, password string) (bool, error) {
  if username == "admin" && password == "admin" {
    return true, nil
  }
  return false, nil
}
```

和上一课时中 UserService 的实现对比，我们会发现 UserService 不再以 LoginRequest 和 LoginResponse 作为输入参数和输出对象，这是因为 Go-kit 的分层理念，认为 Request 和 Response 代表请求和响应，应该是 Transport 层和 Endpoint 层处理的概念，而 Service 层只需要处理最为纯粹的业务逻辑即可，不需要感知或了解请求和响应这些概念。

2. 定义 Endpoint，提供参数转换能力
接下来我们需要建立对应的 Endpoint。它应该是将请求转发给上述的 UserService 处理，然后定义编解码函数 DecodeRequest 和 EncodeResponse，具体代码如下所示：

```
func MakeUserEndpoint(svc UserService) endpoint.Endpoint {
  return func(ctx context.Context, form interface{}) (result interface{}, err error) {
    req := form.(LoginForm)
    ret, err := svc.CheckPassword(ctx,req.Username, req.Password)
    return LoginResult{Ret: ret, Err: err}, nil
  }
}

type LoginForm struct {
  Username           string `json:"username"`
  Password           string `json:"password"`
}

type LoginResult struct {
  Ret bool `json:"ret"`
  Err  error  `json:"err"`
}
```

同样的，Endpoint 层也未直接处理 LoginRequest 和 LoginResponse，而是直接处理 LoginForm 和 LoginResult，使用 DecodeLoginRequest 函数将 LoginRequest 转换成 LoginForm，然后使用 EncodeLoginResponse 将 LoginResult 转换为 LoginResponse。转换函数的具体定义如下所示（这两个函数实际也可以在 Transport 层，为了讲解思路流畅，才放在本处）：

```
func DecodeLoginRequest(ctx context.Context, r interface{}) (interface{}, error) {
  req := r.(*pb.LoginRequest)
  return LoginForm{
    Username:      req.Username,
    Password:      req.Password,
  }, nil
}
​
​
func EncodeLoginResponse(_ context.Context, r interface{}) (interface{}, error) {
  resp := r.(LoginResult)
  retStr := "fail"
  if resp.Ret {
    retStr = "success"
  }
  errStr := ""
  if resp.Err != nil {
    errStr = resp.Err.Error()
  }
  return &pb.LoginResponse{
    Ret: retStr,
    Err: errStr,
  }, nil
}
```

这样做的好处有两点：一是 LoginRequest 和 LoginResponse 是通过 gRPC 生成的，属于 Transport 层，Endpoint 层不需要感知，后续如果技术选型变化了，需要将 gRPC 替换成 Thrift 时就可以只处理 Transport 层的变化，让变更最小化（如下图）；二是后端业务处理时的属性类型和返回给前端的数据属性类型不一定完全一样，比如上述代码中 LoginResult 中的 Ret 是 bool 类型，而返回给前端的 LoginResponse 中 Ret 是 string 类型，从而实现兼容性。

![](../../../images/go/microservice/service-53.png)

Go-kit 分层示意图

如上图所示，Service 在最内层，Endpoint 在中间，Transport在最外侧，所以 Transport 是最容易进行变更的一层，越往内层逻辑应该越贴近领域逻辑。

3. 定义 Middleware，提供限流和日志中间件
如上文所说，Endpoint 层可以添加诸如日志、限流、熔断、链路追踪和服务监控等能力，这里我们就以限流为例，讲述如何为 Endpoint 添加额外能力。

```
var ErrLimitExceed = errors.New("Rate limit exceed!")

// 使用x/time/rate创建限流中间件
func NewTokenBucketLimitterWithBuildIn(bkt *rate.Limiter) endpoint.Middleware {
  return func(next endpoint.Endpoint) endpoint.Endpoint {
    return func(ctx context.Context, request interface{}) (response interface{}, err error) {
      // 如果超过流量，就直接返回限流异常
      if !bkt.Allow() {
        return nil, ErrLimitExceed
      }
      return next(ctx, request)
    }
  }
}

// 使用时的代码实例
ratebucket := rate.NewLimiter(rate.Every(time.Second * 100), 1000)
endpoint = user.NewTokenBucketLimitterWithBuildIn(ratebucket)(endpoint)
```

上述代码使用了 x/time/rate 来进行限流，具体则使用了令牌桶限流策略，其中 NewLimiter 函数会生成 Limiter 限流器，有两个参数，一个表示每秒生成多少令牌，另一个表示允许缓存多少令牌。

当请求通过 Endpoint 时，就会被该 Middleware 拦截，然后调用 Limiter 的 Allow 函数，如果当前还存有令牌，则消耗一枚令牌，放行请求，返回 true；如果不存在，则阻拦请求，返回 false。有关令牌桶的限流策略，如果你感兴趣的话，可以自行搜索学习。

除了限流外，Endpoint 的 Middleware 还可以和 Hystrix 结合提供熔断能力，和 ZipkinTracer 结合提供服务链路追踪能力、自定义接口调用统计指标或打印日志。

由此可以看出 Endpoint 的 Middleware 确实是 Go-kit 的核心，众多服务治理相关中间件的集成都使用该层进行封装，提供统一的类似于切面的能力供开发者使用，免去了开发者处理架构集成的烦恼。

4. 定义 Transport，提供网络传输能力
下面，我们来具体看一下 Transport 层的实现。我们要给出 proto 文件中 UserServiceServer 的具体实现，也就是下述代码中的 grpcServer 结构体，它实现了 CheckPassword 方法，其中调用了其成员变量 checkPassword 的 ServeGRPC 方法。

```
type grpcServer struct {
  checkPassword grpc.Handler
}
​
func (s *grpcServer) CheckPassword(ctx context.Context, r *pb.LoginRequest) (*pb.LoginResponse, error) {
  _, resp, err := s.checkPassword.ServeGRPC(ctx, r)
  if err != nil {
    return nil, err
  }
  return resp.(*pb.LoginResponse), nil
}
​
​
func NewUserServer(ctx context.Context, endpoints Endpoints) pb.UserServiceServer {
  return &grpcServer{
    checkPassword: grpc.NewServer(
      endpoints.UserEndpoint,
      DecodeLoginRequest,
      EncodeLoginResponse,
    ),
  }
}
```

checkPassword 的类型是 grpc.Handler，在 NewUserServer 方法中我们可以看到调用 grpc.NewServer 将其创建出来，需要传入 Endpoint 和编解码函数。这也正对应上文原理解析中 Go-kit 过程调用示意图的内容。

5. 启动服务端，注册RPC服务
我们来将上述部分组合起来，建立真正的网络服务端，并注册对应的 RPC 服务。具体代码如下所示：

```
func main() {
  flag.Parse()

  ctx := context.Background()
  // 建立 service
  var svc user.UserService
  svc = user.UserServiceImpl{}
  // 建立 endpoint
  endpoint := user.MakeUserEndpoint(svc)
  // 构造限流中间件
  ratebucket := rate.NewLimiter(rate.Every(time.Second * 1), 100)
  endpoint = user.NewTokenBucketLimitterWithBuildIn(ratebucket)(endpoint)
  
  endpts := user.Endpoints{
    UserEndpoint:      endpoint,
  }
  // 使用 transport 构造 UserServiceServer
  handler := user.NewUserServer(ctx, endpts)
  // 监听端口，建立 gRPC 网络服务器，注册 RPC 服务
  ls, _ := net.Listen("tcp", "127.0.0.1:8080")
  gRPCServer := grpc.NewServer()
  pb.RegisterUserServiceServer(gRPCServer, handler)
  gRPCServer.Serve(ls)

}
```

在 main 函数中，首先创建 UserService；然后调用 MakeUserEndpoint 函数创建 Endpoint，并使用限流中间件封装；接着调用 user 的 NewUserServer 方法，传入对应的 Endpoint，得到对应的 gRPC 处理器（Handler）；最后监听网络端口，创建 gRPC 服务端，并注册对应的处理器，即可启动 gRPC 服务端。

至此，我们就构造了一个基于 gRPC 来提供 RPC 通信能力的 Go-kit 项目。

