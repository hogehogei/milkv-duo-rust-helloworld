# milkv-duo-rust-helloworld

- 作業環境  
  Windows 10 WSL2 上で確認  
  Linux DESKTOP-XXXXXXX 5.15.133.1-microsoft-standard-WSL2 #1 SMP Thu Oct 5 21:02:42 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
  
- ライブラリの準備  
  下記のSDKをビルド環境上の任意の場所に置いておく  
  https://github.com/milkv-duo/duo-sdk/tree/master

- Rust設定  
  ~/.cargo/config.toml に以下の設定を追加  
  パスを指定している部分は、上で設定した duo-sdk のディレクトリに置き換えて指定すること  
  ```
  [target.riscv64gc-unknown-linux-musl]
  linker = "/home/hogehogei/milkv-duo/duo-sdk/riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl-gcc"
  rustflags = [
      "-C", "target-feature=-crt-static",
      "-C", "link-arg=--sysroot=/home/hogehogei/milkv-duo/duo-sdk/riscv64-linux-musl-x86_64/sysroot",
      # "-C", "target-feature=+crt-static", # Uncomment me to force static compilation
      # "-C", "panic=abort", # Uncomment me to avoid compiling in panics
  ]
  ```

- ビルドコマンド  
  ```
  $ cargo +nightly build --target riscv64gc-unknown-linux-musl -Zbuild-std --release
  ```

- Milk-V Duo にコピーしてエラーになるとき  
  以下のエラーが出る場合  
  ```
  [root@milkv-duo] : # ./milkv-duo-rust-helloworld
  -sh: milkv-duo-rust-helloworld: not found
  ```
  ビルド環境上で readelf コマンドを使用して、リンクされているライブラリを調べる
  ```
  $ readelf -l target/riscv64gc-unknown-linux-musl/release/milkv-duo-rust-helloworld
  
  ...
    INTERP       0x0000000000000238 0x0000000000000238 0x0000000000000238
                 0x0000000000000020 0x0000000000000020  R      0x1
      [Requesting program interpreter: /lib/ld-musl-riscv64xthead.so.1]
  ```
  Milk-V Duo の環境でlnでリンクを追加すると実行できる
  ```
  [root@milkv-duo] : # find /lib -name "**ld-musl**"
  /lib/ld-musl-riscv64v0p7_xthead.so.1
  [root@milkv-duo] : # ln -sf /lib/ld-musl-riscv64v0p7_xthead.so.1 /lib/ld-musl-riscv64xthead.so.1
  ```

- 参考URL  
  基本下記URLの方法に従って実施。SDKの部分だけ Milk-V Duo の公式ビルドのものを使用するようにした。  
  https://barretts.club/posts/i-got-a-milkv-duo/
