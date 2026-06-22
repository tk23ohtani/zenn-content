---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 #2
emoji: ⚙
type: tech
topics:
  - TDD
  - Rust
published: false
---

TODO Listsを確認しておこう。

```markdown
- [ ] $5 + 10 CHF = $10
- [x] $5 * 2 = $10
- [ ] amountのprivateにする
- [ ] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
```

Dollarオブジェクトを操作すると値が変わってしまう。

```rust:lib.rs
#[cfg(test)]
mod tests {
    // ...
    #[test]
    fn test_multiplication() {
        let mut five = Dollar::new(5);
        five.times(2);
        assert_eq!(10, five.amount);
        five.times(3);  // $10 * 3 になる！
        assert_eq!(15, five.amount);
    }
}
```

Redだ。

```text
thread 'tests::test_multiplication' (3406) panicked at src/lib.rs:32:9:
assertion `left == right` failed
  left: 15
 right: 30
```

元の$5に何度でもかけ算が出来るようテストを書き換える。

```rust:lib.rs
#[cfg(test)]
mod tests {
    use crate::Dollar;
    #[test]
    fn test_multiplication() {
        let mut five = Dollar::new(5);
        let mut product = five.times(2);
        assert_eq!(10, product.amount);
        product = five.times(3);
        assert_eq!(15, product.amount);
    }
}
```

Dollarのtimesメソッドの定義を変更しないとビルドできない。Rustではnullを返せないので、零で初期化したDollarを返すことにした。

```rust:lib.rs
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) -> Dollar {
        self.amount *= _multiplier;
        Dollar { amount: 0 } // null の代わり
    }
    fn equals(&self, _other: &Self) {}
}
```

テストが失敗すりょうになったので、正しい金額を返すDollarを返すようにする。

```rust:lib.rs
impl Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) -> Dollar {
        Dollar {
            amount: self.amount * _multiplier,
        }
    }
    fn equals(&self, _other: &Self) {}
}
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

