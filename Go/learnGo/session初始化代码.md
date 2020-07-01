此种方式仅适用于gin框架，其他框架暂时不知道^_^

```go
import (
	"github.com/gin-contrib/sessions"
	"github.com/gin-contrib/sessions/cookie"
	"github.com/gin-gonic/gin"
	"blog-go-gin/config"
)

func Session(keyPairs string) gin.HandlerFunc {
	store := SessionConfig()
	return sessions.Sessions(keyPairs, store)
}
func SessionConfig() sessions.Store {
	sessionMaxAge := 3600
	sessionSecret := config.GetCfg().AppKey
	var store sessions.Store
	store = cookie.NewStore([]byte(sessionSecret))
	store.Options(sessions.Options{
		MaxAge: sessionMaxAge, //seconds
		Path:   "/",
	})
	return store
}
```



```go
//调用时

session := sessions.Default(c)
byteUser, _ := json.Marshal(user)
session.Set("admin_user", "234544")
session.Get("admin_user")
session.Del("admin_user")
_ = session.Save()           // 一定要加这个
```

