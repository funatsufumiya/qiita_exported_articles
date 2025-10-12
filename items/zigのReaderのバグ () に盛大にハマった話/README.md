## 先に結論

- ファイルから大きなバイト列の固まりを読もうとして `reader.read()` で失敗するときは、`file.read()` を試してみると良い。
- `StreamSource` でファイルをラップするのは、現状（Zig 0.13.0）では避けるべきかもしれない。

:::note warn
以下、実際の原因については不明で未検証です。これが原因かな？と本文中に書いていたとしても、もっと複雑な要因によるものかもしれませんし、将来のアップデートで改善される可能性もあります。
:::

## 概要

Zig 0.13.0において、おそらくバグの一種ではないかと思うのだけれど、ファイルおよびメモリのバイトの読み書きについてのコードを書いていたときに、盛大にハマった。

例えば、以下のようなコードはよく書くのではないかと思う。

```zig
const std = @import("std");

pub fn main() !void {
    const file = try std.fs.cwd().openFile("test.bin", .{});
    defer file.close();

    const reader = file.reader();

    const a = try reader.readInt(i32, .little);
    const b = try reader.readInt(i32, .little);
    const c = try reader.readInt(i32, .little);


    const bytes = try reader.readBytesNoEof(1000);

    std.debug.print("a: {}, b: {}, c: {}\n", .{ a, b, c });

    std.debug.print("bytes.len: {}\n", .{bytes.len});
}

// a: 335544320, b: 1887007846, c: 538997873
// bytes.len: 1000
```

これ自体は特に何も問題ない。ただ、ここにファイルのシークと、`read()` が加わってくると話が違ってくる。

```zig
const std = @import("std");

pub fn main() !void {
    const file = try std.fs.cwd().openFile("test.bin", .{});
    defer file.close();

    const reader = file.reader();

    const a = try reader.readInt(i32, .little);
    const b = try reader.readInt(i32, .little);
    const c = try reader.readInt(i32, .little);

    std.debug.print("a: {}, b: {}, c: {}\n", .{ a, b, c });

    const end = try file.getEndPos();

    try file.seekTo(end - 10);

    const d = try reader.readInt(i32, .little);
    std.debug.print("d: {}\n", .{d});

    try file.seekTo(0);

    const allocator = std.heap.page_allocator;
    const bytes = try allocator.alloc(u8, 1000);
    defer allocator.free(bytes);

    const size = try reader.read(bytes); // <-- !! 注意の必要な箇所 !!

    std.debug.print("bytes.len: {}\n", .{bytes.len});
    std.debug.print("size: {}\n", .{size});
}

// a: 335544320, b: 1887007846, c: 538997873
// d: 842675250
// bytes.len: 1000
// size: 1000
```

この例では実際に問題は起きていないものの、これと同様のコードを書き進めていたところ、 `try reader.read(bytes);` で実際にはバイト列が読めないというエラー（あるいは応答がフリーズするなど）が多々発生した。

（ `try reader.readAllAlloc()` 等のAlloc系はさらに多くのエラー事例が報告されていて、GitHub Issueでいくつか報告を見ることができる。原因は様々。）

自分はこれで半日近くハマってしまったのだけれど、実は、`reader.read(bytes)` を `file.read(bytes)` に単純に置き換えたところ、問題が解消した。

```zig
// const size = try reader.read(bytes); // これがダメなときは
const size = try file.read(bytes); // こうすると動く（かもしれない）

std.debug.print("bytes.len: {}\n", .{bytes.len});
std.debug.print("size: {}\n", .{size});
```

原因やどういう条件下で発生しているかは不明なものの、今のところ `file.read()` では確実に成功するのでそういうものなのだと理解している。

（なお、`reader.readInt` 等では同様のエラーに出くわしたことはないので、読み込むバイト数の大きさ等にも依るのかもしれない。）

## StreamSourceでさらにハマる

実は、上記でハマるさらに前に、`StreamSource`を使っていてハマってしまっていたのでこちらも一応メモしておきたい。

[以下の記事](https://zenn.dev/tetsu_koba/articles/1e3832a9b5b247)をみて、「読み書きする対象としてファイルとメモリ中のバッファとを統一的に扱いたい」ときは、StreamSourceを使えば良いのだなと知った。

https://zenn.dev/tetsu_koba/articles/1e3832a9b5b247

そして、ファイルとメモリを統一的に扱いたいニーズがあったの前述の例を以下のように書き換えた。

```zig
const std = @import("std");

pub fn main() !void {
    const file = try std.fs.cwd().openFile("test.bin", .{});
    defer file.close();

    var source = std.io.StreamSource{ .file = file };
    // ↓  file.reader() ではない点に注意。file自体には以降アクセスさせない。
    const reader = source.reader();

    const a = try reader.readInt(i32, .little);
    const b = try reader.readInt(i32, .little);
    const c = try reader.readInt(i32, .little);

    std.debug.print("a: {}, b: {}, c: {}\n", .{ a, b, c });

    const end = try file.getEndPos();

    try source.seekTo(end - 10);

    const d = try reader.readInt(i32, .little);
    std.debug.print("d: {}\n", .{d});

    try source.seekTo(0);

    const allocator = std.heap.page_allocator;
    const bytes = try allocator.alloc(u8, 1000);
    defer allocator.free(bytes);

    const size = try reader.read(bytes); // <-- !! 注意の必要な箇所 !!

    std.debug.print("bytes.len: {}\n", .{bytes.len});
    std.debug.print("size: {}\n", .{size});
}
// a: 335544320, b: 1887007846, c: 538997873
// d: 842675250
// bytes.len: 1000
// size: 1000
```

これまた例のなかでは特に問題は起きていないのだけれど、やはり、実際に書き進めていくと同じく `try reader.read(bytes);` が失敗するケースに出くわした。しかも、毎回のように起こるエラーが違っていて大混乱。

StreamSourceなら、以下のようにファイルに限らずメモリの読み書きもできるので、便利で良いと考えていたのに、困った事態になった。（※ ただし、問題が起きたのはファイル読み込み時のみで、メモリ読み込み時は問題は発生しないことには注意。）

```zig
var source = std.io.StreamSource{ .const_buffer = std.io.fixedBufferStream(buf) };
// var source = std.io.StreamSource{ .buffer = std.io.fixedBufferStream(buf_allocated) };
```

さらに困ったことに、Fileの場合と違って、StreamSourceから直接read（つまり `source.read(bytes)` ）しようとすると、`NotOpenForReading` エラーが出るという事態になった[^2]。

`NotOpenForReading`についてはさておき、おそらく、`reader.read()` ができなかった理由は、読み込むバイト数が多かったりなど特定の条件下で発生する問題ではないかと感じているのだけれど、結局埒が明かなかったので、StreamSourceでラップするのをやめて、例えば以下のように、シンプルに条件分岐するコードを書くことにした。

```zig
fn readInt(self: *MyClazz, comptime T: type, endian: std.builtin.Endian) !T {
    if (self.stream_reader) |reader| {
        const bytes = try reader.readInt(T, endian);
        return @as(T, @bitCast(bytes));
    } else if (self.file_reader) |reader| {
        const bytes = try reader.readInt(T, endian);
        return @as(T, @bitCast(bytes));
    }
    unreachable;
}

fn read(self: *MyClazz, buffer: []u8) !usize {
    if (self.stream_reader) |reader| {
        return try reader.read(buffer);
    } else if (self.file) |file| { // ※ readerではない点に注意。
        return try file.read(buffer);
    }
    unreachable;
}

// 他の関数も同様。割愛。
```

もっと良い書き方はあるはずだと思うし、あまり美しくはないのだけれど、これで問題は解消され、一応、メモリからのreadとファイルからのreadを両方達成できるようにはなった。

## まとめ

Zigはまだまだ発展途上の言語というのもあり、まだ全般的にドキュメントが不足していたり、破壊的変更が多かったり、バグと思われる事例に出くわすことがある。

実際にバグなのかどうかはわからないし[^1]、使用事例も多くないので一般的にはこう書くべきというコード自体もよくわかっていないのだけれど、こうした問題は言語の発展とともに少しずつ解消されていくのではないかと思う。

[^2]: 今思うと、StreamSourceは内部にfileを保持しているので、条件分岐でこのfileにアクセスして`read()`するという手もあったかもしれない。未検証。（`NotOpenForReading`の原因についても未検証。）

[^1]: 本当にバグだったとしたら、問題の起こるバイナリファイルなどをもっと特定して、バグ報告を上げるべきかもしれない。これについては今後検討していきたい
