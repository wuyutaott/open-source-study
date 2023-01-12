## 项目地址
https://github.com/zsly3n3/gameServerForLeaf

### 知识点列表
- 微信登录功能
    * openid的获取，GetOpenID，登录的时候使用
- 雪花算法，分布式id
- 二维码相关算法
- 消息采用json解析
- UUID的生成算法
- 一些玩家匹配逻辑

### 知识点 - 01
批量建表
```go
func resetDB(engine *xorm.Engine){
    user := &datastruct.User{}
    robotName := &datastruct.RobotName{}
    maxScoreInEndlessMode := &datastruct.MaxScoreInEndlessMode{}
    maxScoreInSinglePersonMode := &datastruct.MaxScoreInSinglePersonMode{}
    maxScoreInInviteMode := &datastruct.MaxScoreInInviteMode{}
    skinFragment := &datastruct.SkinFragment{}
    gameIntegral := &datastruct.GameIntegral{}
    err := engine.DropTables(user,robotName,maxScoreInEndlessMode,maxScoreInSinglePersonMode,maxScoreInInviteMode,skinFragment,gameIntegral)
    errhandle(err)
	err = engine.CreateTables(user,robotName,maxScoreInEndlessMode,maxScoreInSinglePersonMode,maxScoreInInviteMode,skinFragment,gameIntegral)
    errhandle(err)
}
```

### 知识点 - 02
定时任务
```go
// 看起来像是一个定时统计分数的功能
func timerTask(engine *xorm.Engine){
    c := cron.New()
	spec :="0 0 0 * * 1"
    c.AddFunc(spec, func() {
        table0 := "max_score_in_single_person_mode"
        table1 := "max_score_in_endless_mode"
        truncate := "truncate table "
        engine.Exec(truncate+table0)
        engine.Exec(truncate+table1)
    })
    c.Start()
}
```

### 知识点 - 03
生成UUID
```go
import (
    crypto_rand "crypto/rand"
)

func UniqueId() string {    
    b := make([]byte, 48)  
    if _, err := io.ReadFull(crypto_rand.Reader, b); err != nil {  
        return ""
    }  
    return getMd5String(base64.URLEncoding.EncodeToString(b))  
}
```