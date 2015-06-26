# Pingo: Plugins for Go

Pingo 是一个用来为 Go 程序编写插件的简单独立库，因为 Go 本身是静态链接的，因此所有插件都以外部进程方式存在。

Pingo 旨在简化标准 RPC 包，支持 TCP 和 Unix 套接字作为通讯协议。

当前还不支持远程插件，如果有需要，远程插件很快会提供。


## Example

使用 Pingo 创建一个插件非常简单，首先新建目录，如 "plugins/hello-world" ，然后在该目录下编写 main.go:

```go
// Always create a new binary
package main

import "github.com/dullgiulio/pingo"

// Create an object to be exported
type MyPlugin struct{}

// Exported method, with a RPC signature
func (p *MyPlugin) SayHello(name string, msg *string) error {
    *msg = "Hello, " + name
    return nil
}

func main() {
	plugin := &MyPlugin{}

	// Register the objects to be exported
	pingo.Register(plugin)
	// Run the main events handler
	pingo.Run()
}
```

编译:
```sh
$ cd plugins/hello-world
$ go build
```

You should get an executable called "hello-world". Congratulations, this is your plugin.

Now, time to use the newly create plugin.

In your main executable, invoke the plugin you have just created:

```go
package main

import (
	"log"
	"github.com/dullgiulio/pingo"
)

func main() {
	// Make a new plugin from the executable we created. Connect to it via TCP
	p := pingo.NewPlugin("tcp", "plugins/hello-world/hello-world")
	// Actually start the plugin
	p.Start()
	// Remember to stop the plugin when done using it
	defer p.Stop()

	var resp string

	// Call a function from the object we created previously
	if err := p.Call("MyPlugin.SayHello", "Go developer", &resp); err != nil {
		log.Print(err)
	} else {
		log.Print(resp)
	}
}
```

Now, build your executable and all should work!  Remember to use the correct path to
your plugins when you make the Plugin object.  Ideally, always pass an absolute path.

## Bugs

Report bugs in Github.  Pull requests are welcome!

## License

MIT
