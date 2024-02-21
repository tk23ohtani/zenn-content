---
title: "Win32API Log Output"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [markdown]
published: false
---

Win32APIを使ってログ出力機能を持つWindowsデスクトップアプリケーションをC言語で作成する例

- `dprintf.h`

```h
#pragma once

#ifdef __cplusplus
extern "C" {
#endif

	void debug_printf(const char* format, ...);

#ifdef __cplusplus
}
#endif
```

- `dprintf.c`

```c
#include <Windows.h>
#include <stdio.h>
#include <stdarg.h>

#include "dprintf.h"

void debug_printf(const char* format, ...)
{
    char buffer[1024];
    va_list args;
    va_start(args, format);
    vsnprintf(buffer, sizeof(buffer), format, args);
    va_end(args);

    OutputDebugStringA(buffer);
}
```

- 使用例

```c
#include "dprintf.h"

int main()
{
    debug_printf("Hello, %s!\n", "world");
    debug_printf("The answer is %d.\n", 42);

    return 0;
}
```

この例では、`debug_printf`関数を使って、`printf()`と同じ書式指定子を使ったログ出力する。`OutputDebugStringA`関数を使って、デバッグ出力ウィンドウにログが出力される。Visual Studioなどのデバッガを使用している場合、デバッグ出力ウィンドウでログを確認できる。実行ファイルを直接実行した場合、ログは出力されないので、ログを確認するには、DebugView (https://docs.microsoft.com/en-us/sysinternals/downloads/debugview) などのツールを使用する。

