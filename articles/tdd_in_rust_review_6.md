---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 #6
emoji: ⚙
type: tech
topics:
  - TDD
  - Rust
published: false
---

TODO Listsを確認しておこう。

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

共通の親クラスを作成する。

```rust:lib.rs
pub struct Money {}
```

Rustには継承ががないので、代わりにTraitを使う。

```rust:lib.rs
trait MoneyTrait {
}
```

DollarにTraitを適用する。

```rust:lib.rs
impl MoneyTrait for Dollar {
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

宣言がいないとビルドできないので書き足す。

```rust:lib.rs
trait MoneyTrait {
    fn new(_amount: i32) -> Self;
    fn times(&mut self, _multiplier: i32) -> Self;
    fn equals(&self, _other: Self) -> bool;
}
```

equlsメソッドの親クラスに移動する「メソッドの引き上げ」に着手する。

```rust:lib.rs
pub struct Money {
    amount: i32,
}

impl Money {
    fn new(_amount: i32) -> Self {
        Money { amount: _amount }
    }
}
```

```rust:lib.rs
// 無理矢理変換する
impl Dollar {
    fn get_money(&self) -> Money {
        Money::new(self.amount)
    }
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

