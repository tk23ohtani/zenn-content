---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 #5
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
- [x] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
- [ ] 5 CHF * 2 = 10 CHF
```

DollarをコピーしてFrancのテストを書いてみる。

```rust:lib.rs
#[cfg(test)]
mod tests {
    use crate::Dollar;

    #[test]
    fn test_multiplication() {
        let mut five = Dollar::new(5);
        assert!(Dollar::new(10).equals(five.times(2)));
        assert!(Dollar::new(15).equals(five.times(3)));
    }

    #[test]
    fn test_equality() {
        assert!(Dollar::new(5).equals(Dollar::new(5)));
        assert!(!Dollar::new(5).equals(Dollar::new(6)));
    }

    #[test]
    fn test_franc_multiplication() {
        let mut five = Franc::new(5);
        assert!(Franc::new(10).equals(five.times(2)));
        assert!(Franc::new(15).equals(five.times(3)));
    }
}
```

ここはコピペ駆動。

```rust:lib.rs
#[derive(Debug)]
pub struct Franc {
    amount: i32,
}

impl Franc {
    fn new(_amount: i32) -> Franc {
        Franc { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) -> Franc {
        Franc {
            amount: self.amount * _multiplier,
        }
    }
    fn equals(&self, _other: Franc) -> bool {
        self.amount == _other.amount
    }
}

impl PartialEq for Franc {
    fn eq(&self, _other: &Franc) -> bool {
        self.amount == _other.amount
    }
}
```

宣言追加。

```rust:lib.rs
#[cfg(test)]
mod tests {
    use crate::Dollar;
    use crate::Franc;
    // ...
}
```

amountはもともとprivateだった。

```markdown
- [x] $5 * 2 = $10
- [x] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
- [x] 5 CHF * 2 = 10 CHF
- [ ] DollarとFrancの重複
- [ ] equalsの一般化
- [ ] timesの一般化
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

