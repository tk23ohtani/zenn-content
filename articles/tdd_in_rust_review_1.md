---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 #1
emoji: ⚙
type: tech
topics:
  - TDD
  - Rust
published: false
---

Dev Containerを使った開発環境を利用する。

実際にテストが通るかどうか。
自動的に組み込まれるライブラリコードに実装していくものとする。



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



```text
running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

TODO Lists

```markdown
- [ ] $5 + 10 CHF = $10
- [ ] $5 * 2 = $10
```

最初のテスト。

```rust
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

```markdown
- [ ] $5 + 10 CHF = $10
- [ ] $5 * 2 = $10
- [ ] amountのprivateにする
- [ ] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
```

まず、`Dollar`クラス相当なものを作る。
```rust
struct Dollar {}
```

次に、コンストラクタ相当なものを作る。
```rust
impl Dollar {
    fn new(amount: i32) -> Self {}
}
```

times()メソッドを作る。
```rust
impl Dollar {
    fn new(amount: i32) -> Self {}
    fn times(&self, _multiplier: i32) -> Self {}
}
```

equals()メソッドを作る。
```rust
impl Dollar {
    fn new(amount: i32) -> Self {}
    fn times(&self, _multiplier: i32) -> Self {}
    fn equals(&self, _other: &Self) -> bool {}
}
```

amountフィールドを作る。

```rust
struct Dollar {
    amount: i32,
}
```

ビルドエラーを解消。

```rust
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: 0 }
    }
    fn times(&self, _multiplier: i32) {}
    fn equals(&self, _other: &Self) {}
}
```

テストが実行できるようになった。

```test
thread 'tests::test_multiplication' (22347) panicked at src/lib.rs:28:9:
assertion `left == right` failed
  left: 10
 right: 0
```

テストはRedなので、とりあえずGreenになるようにする。

```rust
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: 10 }
    }
    fn times(&self, _multiplier: i32) {}
    fn equals(&self, _other: &Self) {}
}
```

マジックナンバーを解消する。

```rust
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: 5 * 2 }
    }
    fn times(&self, _multiplier: i32) {}
    fn equals(&self, _other: &Self) {}
}
```

一手で解消できないので、times()メソッドに移動。この時、selfそのものを変更するので、変更可能な状態で渡す必要がある。

```rust
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: 0 }
    }
    fn times(&mut self, _multiplier: i32) {
        self.amount = 5 * 2
    }
    fn equals(&self, _other: &Self) {}
}
```

Rustの制約によりエラーになるので、テストを修正。

```rust
#[cfg(test)]
mod tests {
    use crate::Dollar;
    #[test]
    fn test_multiplication() {
        let mut five = Dollar::new(5);
        five.times(2);
        assert_eq!(10, five.amount);
    }
}
```

```rust
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) {
        self.amount = 5 * 2
    }
    fn equals(&self, _other: &Self) {}
}
```

```rust
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) {
        self.amount = self.amount * 2
    }
    fn equals(&self, _other: &Self) {}
}
```




```rust
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) {
        self.amount = self.amount * _multiplier
    }
    fn equals(&self, _other: &Self) {}
}
```


```rust
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) {
        self.amount *= _multiplier
    }
    fn equals(&self, _other: &Self) {}
}
```

```rust
```

```rust
```

```rust
```

```rust
```

```rust
```

```rust
```

```rust
```





