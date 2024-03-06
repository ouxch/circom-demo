###### 安装 circom 编译器和 snarkjs

circom 编译器是 rust 语言编写的，需先安装 rust

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

从源码编译安装 circom

```shell
git clone https://github.com/iden3/circom.git
cd circom
cargo build --release
cargo install --path circom # circom 二进制文件会安装到 $HOME/.cargo/bin 目录下 
```

snarkjs 是 circom 团队分发的 npm 软件包，可基于 circom 输出的文件生成零知识证明以及验证证明。

```shell
npm install -g snarkjs
```

###### example

编译 circom 算术电路，得到算术方程系统

```shell
circom multiplier2.circom --r1cs --wasm --sym --c
```

计算见证(witness)，CPP 代码还不能在ARM架构上运行，这里使用 WebAssembly 生成

```shell
cd multiplier2_js
node generate_witness.js multiplier2.wasm input.json witness.wtns
cd ..
```

初始化可信设置

```shell
# 跟电路无关的部分
snarkjs powersoftau new bn128 12 pot12_0000.ptau -v
snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
# 跟电路有关的部分
snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v
snarkjs groth16 setup multiplier2.r1cs pot12_final.ptau multiplier2_0000.zkey
snarkjs zkey contribute multiplier2_0000.zkey multiplier2_0001.zkey --name="1st Contributor Name" -v
snarkjs zkey export verificationkey multiplier2_0001.zkey verification_key.json
```

生成证明，证明包含两个文件：

- proof.json 包含了证明
- public.json 包含公共输入和输出的值

```shell
snarkjs groth16 prove multiplier2_0001.zkey multiplier2_js/witness.wtns proof.json public.json
```

验证证明：命令输出OK，证明有效

```shell
snarkjs groth16 verify verification_key.json public.json proof.json
```
