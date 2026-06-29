---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §1
emoji: ⚙
type: tech
topics:
  - TDD
  - Rust
published: false
---

前提：Dev Containerを使った開発環境を利用する。

実際にテストが通るかどうか。  
なお、自動的に組み込まれるライブラリコードに実装していくものとする。  

```rust:lib.rs
// TDD in Rust のライブラリコード

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

テストが成功していればOK!

```text
running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

TODO Listsを確認しておこう。

```markdown
- [ ] $5 + 10 CHF = $10
- [ ] $5 * 2 = $10
```

最初のテストを作る。

```rust:lib.rs
#[cfg(test)]
mod tests {
    use crate::Dollar;
    #[test]
    fn test_multiplication() {
        let five = Dollar::new(5);
        five.times(2);
        assert_eq!(10, five.amount);
    }
}
```

TODO Listsに追加する。

```markdown
- [ ] $5 + 10 CHF = $10
- [ ] $5 * 2 = $10
- [ ] amountのprivateにする
- [ ] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
```

まず、`Dollar`クラス相当なものを作る。

```rust:lib.rs
struct Dollar {}
```

次に、コンストラクタ相当なものを作る。

```rust:lib.rs
impl Dollar {
    fn new(amount: i32) -> Self {}
}
```

times()メソッドを作る。

```rust:lib.rs
impl Dollar {
    // ...
    fn times(&self, _multiplier: i32) -> Self {}
}
```

equals()メソッドを作る。

```rust:lib.rs
impl Dollar {
    // ...
    fn equals(&self, _other: &Self) -> bool {}
}
```

最後に、amountフィールドを作る。

```rust:lib.rs
struct Dollar {
    amount: i32,
}
```

ここまでの状態ではビルドエラーになるので解消する。

```rust:lib.rs
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: 0 }
    }
    // ...
}
```

ビルドが通ったら、テストが実行できるようになったことを確認する。

```text
thread 'tests::test_multiplication' (22347) panicked at src/lib.rs:28:9:
assertion `left == right` failed
  left: 10
 right: 0
```

テストはRedなので、ひとまず最短でGreenになるようにする。

```rust:lib.rs
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: 10 }
    }
    // ...
}
```

マジックナンバーを解消する。

```rust:lib.rs
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: 5 * 2 }
    }
    // ...
}
```

一手で解消できないので、初期化時からtimes()メソッドに移動。  
この時、selfそのものを変更するので、変更可能な状態で渡す必要がある。

```rust:lib.rs
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: 0 }
    }
    fn times(&mut self, _multiplier: i32) {
        self.amount = 5 * 2
    }
    // ...
}
```

Rustの制約によりエラーになるので、テスト側も修正している。

```rust:lib.rs
#[cfg(test)]
mod tests {
    // ...
    #[test]
    fn test_multiplication() {
        let mut five = Dollar::new(5);
    // ...
    }
}
```

テストから5が渡ってくるので、それをコンストラクタで設定する。

```rust:lib.rs
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) {
        self.amount = 5 * 2
    }
    // ...
}
```

times()メソッドの中でamountを使えばいい。

```rust:lib.rs
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) {
        self.amount = self.amount * 2
    }
    // ...
}
```

テストから渡る引数の値が2だからそれを使える。

```rust:lib.rs
impl Dollar {
    // ...
    fn times(&mut self, _multiplier: i32) {
        self.amount = self.amount * _multiplier
    }
    // ...
}
```

重複排除のため三項演算子を使う。（あくまで重複排除のためｗ）

```rust:lib.rs
impl Dollar {
    // ...
    fn times(&mut self, _multiplier: i32) {
        self.amount *= _multiplier
    }
    // ...
}
```

ここまでで一つだけTODO Listsが更新できる。

```markdown
- [ ] $5 + 10 CHF = $10
- [x] $5 * 2 = $10
- [ ] amountのprivateにする
- [ ] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
```

２章につづく。
