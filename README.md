# gRPC Polyglot Lab

Praktikum komunikasi inter-service dengan gRPC: satu server Go dan dua client (Python dan Node.js) yang memakai satu kontrak `user.proto`.

## Struktur Proyek

```
grpc-polyglot-lab/
├── user.proto        # kontrak API (Single Source of Truth)
├── go-server/        # gRPC server (Go)
├── python-client/    # gRPC client (Python)
└── node-client/      # gRPC client (Node.js)
```

## Cara Menjalankan

1. Terminal 1: `cd go-server && go run main.go`
2. Terminal 2: `cd python-client && python client.py`
3. Terminal 3: `cd node-client && node client.js`

## Analisis: Static Code Generation vs Dynamic Runtime Generation

Pada pendekatan **static code generation** (Go dan Python), file `user.proto` diproses terlebih dahulu oleh compiler `protoc` dengan bantuan plugin (`protoc-gen-go` dan `protoc-gen-go-grpc` untuk Go, `grpc_tools.protoc` untuk Python) menjadi file kode sumber nyata, yaitu `user.pb.go`, `user_grpc.pb.go`, `user_pb2.py`, dan `user_pb2_grpc.py`. File hasil generate ini menjadi bagian dari proyek dan dikompilasi (Go) atau diimpor (Python) seperti kode biasa, sehingga struct, class, dan stub tersedia secara eksplisit, tipe data dicek sejak awal, dan IDE dapat memberi autocomplete. Konsekuensinya, setiap kali `user.proto` berubah (misalnya penambahan field `phone`), developer harus menjalankan ulang `protoc` agar kode hasil generate sinkron dengan kontrak. Sebaliknya, pada pendekatan **dynamic runtime generation** (Node.js), tidak ada langkah kompilasi terpisah: library `@grpc/proto-loader` membaca dan mem-parsing `user.proto` langsung ketika aplikasi dijalankan, lalu membentuk definisi service di memori melalui `grpc.loadPackageDefinition`. Pendekatan ini lebih ringkas dan fleksibel karena perubahan skema langsung terbaca setelah program dijalankan ulang, tetapi tidak ada pengecekan tipe saat kompilasi (kesalahan nama field baru ketahuan saat runtime) dan ada sedikit beban parsing saat startup. Secara singkat, static generation memindahkan pemrosesan skema ke waktu build demi keamanan tipe dan performa, sedangkan dynamic generation memindahkannya ke waktu eksekusi demi kemudahan dan fleksibilitas.
