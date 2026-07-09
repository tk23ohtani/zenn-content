---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §14
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
- [ ] $5 + $5 = $10
- [ ] $5 + $5 がMoneyを返す
- [x] Bank.reduce(Money)
- [ ] Moneyを変換して換算を行う
- [ ] Reduce(Bank, String)
```

2フランを1ドルに換算したい、テストはすぐ書ける。

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_reduce_money_different_currency() {
        let bank = Bank::new();
        bank.addRate("CHF", "USD", 2);
        let result = bank.reduce(Money::franc(2), "USD");
        assert_eq!(Money::dollar(1), result);
    }
}
```

❌addRateメソッドを作る。

```rust:lib.rs
impl Bank {
    // ...
    fn addRate(&self, _from: &'static str, _to: &'static str, _rate: i32) {}
}
```

❌ビルドは通るようになるがテスト波失敗する。Greenにするために2で割る。

```rust:lib.rs
impl Expression for Money {
    fn reduce(&self, _bank: &Bank, _to: &'static str) -> Money {
        let mut rate: i32 = 1;
        if (self.currency() == "CHF") && (_to == "USD") {
            rate = 2;
        }
        Money::new(self.amount / rate, _to)
    }
}
```

✅テストはGreenになった！  
これではMoneyが為替レートを盛ってしまっている。為替レートはBankで一括管理する必要がある。
本ではExpressionにBankを渡す必要がある、が、Rust制約によってすでに実装済みになっている。

```rust:lib.rs
impl Bank {
    // ...
    fn rate(&self, _from: &'static str, _to: &'static str) -> i32 {
        if (_from == "CHF") && (_to == "USD") {
            2
        } else {
            1
        }   
    }
}
```

✅為替レートはBankに問い合わせれば良くなった。  

```rust:lib.rs
impl Expression for Money {
    fn reduce(&self, _bank: &Bank, _to: &'static str) -> Money {
        let rate = _bank.rate(self.currency, _to);
        Money::new(self.amount / rate, _to)
    }
}
```

✅為替レートを表形式でBankの仲に格納していくようだ。

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_array_equals() {
        assert_eq!(["abc"], ["abc"]);
    }
}
```

✅本の実装をまねしてみようと思ったが、これはテストがパスするようだ。
リファクタリングの途中だからテストは書かないそうだ。

```rust:lib.rs
struct Pair {
    from: &'static str,
    to: &'static str,
}

impl Pair {
    fn new(_from: &'static str, _to: &'static str) -> Pair {
        Pair {
            from: _from,
            to: _to,
        }
    }
}
```

実装が続く。

```rust:lib.rs
impl Pair {
    // ...
    fn equals(&self, _object: &Pair) -> bool {
        self.from == _object.from && self.to == _object.to
    }
    fn hashCode() -> i32 {
        0
    }
}
```

ハッシュマップとここから使っていくようだ。

```rust:lib.rs
use std::collections::HashMap;

pub struct Bank {
    rates: HashMap<Pair, i32>,
}

impl Bank {
    fn new() -> Bank {
        Bank {
            rates: HashMap::new(),
        }
    }
    // ...
}
```

```rust:lib.rs
impl Bank {
    // ...
    fn addRate(&mut self, _from: &'static str, _to: &'static str, _rate: i32) {
        self.rates.insert(Pair::new(_from, _to), _rate);
    }
    fn rate(&self, _from: &'static str, _to: &'static str) -> i32 {
        match self.rates.get(&Pair::new(_from, _to)) {
            Some(r) => *r,
            None => 0,
        }
    }
}
```

❌ハッシュのキーに使うために宣言を追加。

```rust:lib.rs
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct Pair {
    from: &'static str,
    to: &'static str,
}
```

```rust:lib.rs
mod tests {
    #[test]
    fn test_identity_rate() {
        assert_eq!(1, Bank::new().rate("USD", "USD"));
    }
}
```

```rust:lib.rs
impl Bank {
    // ...
    fn rate(&self, _from: &'static str, _to: &'static str) -> i32 {
        if _from == _to {
            return 1;
        }
        match self.rates.get(&Pair::new(_from, _to)) {
            Some(r) => *r,
            None => 0,
        }
    }
}
```

✅Greenになった！  
TODOに変更しておこう。

```markdown
- [ ] $5 + 10 CHF = $10
- [x] $5 + $5 = $10
- [ ] $5 + $5 がMoneyを返す
- [x] Bank.reduce(Money)
- [x] Moneyを変換して換算を行う
- [x] Reduce(Bank, String)
```
