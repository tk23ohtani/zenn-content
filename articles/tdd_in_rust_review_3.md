---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §3
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
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [ ] equals()
- [ ] hashCode()
```

まず、$5は$5と等価でなければならない。

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

    #[test]
    fn test_equality() {
        assert!(Dollar::new(5).equals(Dollar::new(5)));
    }
}
```

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
    fn equals(&self, _other: Self) -> bool {
        true
    }
}
```

三角測量のための二つ目の実例テストを追加する。

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

    #[test]
    fn test_equality() {
        assert!(Dollar::new(5).equals(Dollar::new(5)));
        assert!(!Dollar::new(5).equals(Dollar::new(6)));
    }
}
```

等価比較を一般化すｒ。

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
    fn equals(&self, _other: Self) -> bool {
        self.amount == _other.amount
    }
}
```

```markdown
- [ ] $5 + 10 CHF = $10
- [x] $5 * 2 = $10
- [ ] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
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

