```go
package main
import {
	"context"
	"fmt"
	"golang.org/x/sync/errgroup"
	"net/http"
	"os"
	"os/signal"
}
func Httpstart(srv *http.Server) error {
	http.HandleFunc("/hello", HelloServer)
	fmt.Println("http server start")
	err := srv.ListenAndServe()
	return err
}

func HelloServer(w http.ResponseWriter, req *http.Request) {
	io.WriteString(w, "hello, world!\n")
}

func main() {
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)
	group, errCtx := errgroup.WithContext(ctx)
	srv := &http.Server{Addr: ":8080"}

	group.Go(func() error {
		return Httpstart(srv)
	})

	group.Go(func() error {
		<-errCtx.Done()
		fmt.Println("http server stop")
		return srv.Shutdown(errCtx) 
	})

	chanel := make(chan os.Signal, 1) 
	signal.Notify(chanel)

	group.Go(func() error {
		for {
			select {
			case <-errCtx.Done():
				return errCtx.Err()
			case <-chanel:
				cancel()
			}
		}
        return nil
	})

	if err := group.Wait(); err != nil {
		fmt.Println("group error: ", err)
	}
	fmt.Println("all done!")

}

```

