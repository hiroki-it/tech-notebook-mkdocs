---
title: 【知見を記録するサイト】gRPC＠アプリケーション連携
description: gRPC＠アプリケーション連携の知見をまとめました。

---

# gRPC＠アプリケーション連携

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. gRPCの仕組み

![grpc_architecture](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/grpc_architecture.png)

RPCフレームワークの一つで、プロトコルバッファーを用いてRPC（リモートプロシージャーコール）を実行する。RESTful-APIに対するリクエストではリクエストメッセージのヘッダーやボディを作成する必要があるが、リモートプロシージャーコールであれば通信先の関数を指定して引数を渡せばよく、まるで自身の関数のようにコールできる。

参考：

- https://qiita.com/gold-kou/items/a1cc2be6045723e242eb#%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%BA%E3%81%A7%E9%AB%98%E9%80%9F%E5%8C%96
- https://openstandia.jp/oss_info/grpc/

<br>

## 02. Goの場合

### サーバー側

#### ▼ protoファイル

クライアント側で呼び出せるようにする構造体や関数を定義する。

参考：

- https://qiita.com/gold-kou/items/a1cc2be6045723e242eb#%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%BA%E3%81%A7%E9%AB%98%E9%80%9F%E5%8C%96
- https://christina04.hatenablog.com/entry/protoc-usage

```protobuf
// protoファイルの構文のバージョンを設定する。
syntax = "proto3";

// pb.goファイルで自動生成される時のパッケージ名
package foo;

// クライアント側からのリモートプロシージャーコール時に渡す引数を定義する。
// フィールドのタグを1としている。メッセージ内でユニークにする必要があり、フィールドが増えれば別の数字を割り当てる。
message Message {
  string body = 1;
}

// クライアント側からリモートプロシージャーコールされる関数を定義する。
service FooService {
  rpc SayHello(Message) returns (Message) {}
}
```

#### ▼ pb.goファイル

事前に用意した```proto```ファイルを使用して、```pb.go```ファイルを自動生成する。```pb.go```ファイルには、gRPCを使用する上で必要な構造体や関数が定義されており、ユーザーはこのファイルをそのまま使用すれば良い。

参考：

- https://christina04.hatenablog.com/entry/protoc-usage
- https://qiita.com/gold-kou/items/a1cc2be6045723e242eb#%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%BA%E3%81%A7%E9%AB%98%E9%80%9F%E5%8C%96

```bash
$ protoc ./foo/foo.proto --go_out=plugins=grpc:foo 

# foo.pb.goファイルが生成される。
```

ちなみに、pbファイルには、gRPCサーバーとして登録するための```Register*****ServiceServer```関数が定義される。

```go
// 〜 中略 〜

func RegisterFooServiceServer(s *grpc.Server, srv FooServiceServer) {
	s.RegisterService(&_FooService_serviceDesc, srv)
}

// 〜 中略 〜
```

#### ▼ goサーバー

リモートプロシージャーコールを受け付けるサーバーを定義する。サーバーをgRPCサーバーとして登録する必要がある。

参考：

- https://qiita.com/gold-kou/items/a1cc2be6045723e242eb#%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%BA%E3%81%A7%E9%AB%98%E9%80%9F%E5%8C%96
- https://entgo.io/ja/docs/grpc-server-and-client/

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net"

	pb "github.com/hiroki-hasegawa/foo/foo" // pb.goファイルを読み込む。

	"google.golang.org/grpc"
)

// goサーバー
type Server struct {
}

// Helloを返却する関数
func (s *Server) SayHello(ctx context.Context, in *pb.Message) (*Message, error) {
	log.Printf("Received message body from client: %s", in.Body)
	return &pb.Message{Body: "Hello From the Server!"}, nil
}

func main() {

	// goサーバーで待ち受けるポート番号を設定する。
	listenPort, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))

	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	// gRPCサーバーを作成する。
	grpcServer := grpc.NewServer()

	// pb.goファイルで自動生成された関数を用いて、goサーバーをgRPCサーバーとして登録する。
	// goサーバーがリモートプロシージャーコールを受信できるようになる。
	pb.RegisterFooServiceServer(grpcServer, &Server{})

	// gRPCサーバーとして、goサーバーで通信を受信する。
	if err := grpcServer.Serve(listenPort); err != nil {
		log.Fatalf("failed to serve: %s", err)
	}
}
```

<br>

### クライアント側

gRPCサーバーのリモートプロシージャーコールを実行する。

参考：https://qiita.com/gold-kou/items/a1cc2be6045723e242eb#%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%BA%E3%81%A7%E9%AB%98%E9%80%9F%E5%8C%96

```go
package main

import (
	"log"

	"golang.org/x/net/context"
	"google.golang.org/grpc"

	pb "github.com/hiroki-hasegawa/foo/foo" // pb.goファイルを読み込む。
)

func main() {

	// gRPCコネクションを作成する。
	conn, err := grpc.Dial(":9000", grpc.WithInsecure())

	if err != nil {
		log.Fatalf("did not connect: %s", err)
	}

	defer conn.Close()

	// gRPCサーバーとして、goサーバーを作成する。
	c := pb.NewFooServiceClient(conn)

	// goサーバーのリモートプロシージャーコールを実行する。
	response, err := c.SayHello(context.Background(), &pb.Message{Body: "Hello From Client!"})

	if err != nil {
		log.Fatalf("Error when calling SayHello: %s", err)
	}

	log.Printf("Response from server: %s", response.Body)
}
```

<br>