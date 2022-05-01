---
title: "C#のコーディング規約"
last_modified_at: 2022-05-01T09:00:00+09:00
author: teddy-toshi
tags:
  - C#
  - 規約
---

C#では.NETの[C#のコーディング規約](https://docs.microsoft.com/ja-jp/dotnet/csharp/fundamentals/coding-style/coding-conventions)に従って記述するのが一般的です. このコーディング規約はMicrosoftで使用されているものであり, アプリケーション領域を記述するのには厳しすぎる部分があるので, 差分を定義します. また, コーディング規約をサポートする`.gitignore`, `.gitattributes`, `.editorconfig`を記述します.

## 結論. 設定ファイルについて

下記のコマンドで`.gitignore`を作成します.

``` sh
dotnet new gitignore
```

`.gitattributes`に下記を追加します.

```
*.cs text eol=crlf diff=csharp
*.tt text eol=crlf diff=csharp
```

`.editorconfig`に下記を追加します.

```
[*.{cs,tt}]
charset = utf-8-bom
end_of_line = crlf

[*.cs]
#### C# Coding Conventions ####
# var preferences
csharp_style_var_elsewhere = true:suggestion
csharp_style_var_for_built_in_types = true:suggestion
csharp_style_var_when_type_is_apparent = true:suggestion

#### Naming styles #### _exampleIdentifier
# Naming rules
dotnet_naming_rule.private_or_internal_field_should_be_begins_with__.severity = suggestion
dotnet_naming_rule.private_or_internal_field_should_be_begins_with__.symbols = private_or_internal_field
dotnet_naming_rule.private_or_internal_field_should_be_begins_with__.style = begins_with__

# Symbol specifications
dotnet_naming_symbols.private_or_internal_field.applicable_kinds = field
dotnet_naming_symbols.private_or_internal_field.applicable_accessibilities = internal, private, private_protected
dotnet_naming_symbols.private_or_internal_field.required_modifiers =

# Naming styles
dotnet_naming_style.begins_with__.required_prefix = _
dotnet_naming_style.begins_with__.required_suffix =
dotnet_naming_style.begins_with__.word_separator =
dotnet_naming_style.begins_with__.capitalization = camel_case

#### Naming styles #### s_exampleIdentifier
# Naming rules
dotnet_naming_rule.private_or_internal_static_field_should_be_begins_with_s_.severity = suggestion
dotnet_naming_rule.private_or_internal_static_field_should_be_begins_with_s_.symbols = private_or_internal_static_field
dotnet_naming_rule.private_or_internal_static_field_should_be_begins_with_s_.style = begins_with_s_

# Symbol specifications
dotnet_naming_symbols.private_or_internal_static_field.applicable_kinds = field
dotnet_naming_symbols.private_or_internal_static_field.applicable_accessibilities = internal, private, private_protected
dotnet_naming_symbols.private_or_internal_static_field.required_modifiers = static

# Naming styles
dotnet_naming_style.begins_with_s_.required_prefix = s_
dotnet_naming_style.begins_with_s_.required_suffix =
dotnet_naming_style.begins_with_s_.word_separator =
dotnet_naming_style.begins_with_s_.capitalization = camel_case
```

## C#のコーディング規約との差分について

### [暗黙的に型指定されたローカル変数](https://docs.microsoft.com/ja-jp/dotnet/csharp/fundamentals/coding-style/coding-conventions#implicitly-typed-local-variables)を常に使用します

割り当ての右側から型が明らかではない場合でも`var`を使用します. アプリケーションレイヤーではメソッドの戻り値の型で良い場合が多く, 戻り値の型が特定の型である事を必要とする場面はほぼありません. もしそのような場面に遭遇したときは, インターフェースが適切か見直してください.

```cs
var var3 = Convert.ToInt32(Console.ReadLine()); 
var var4 = ExampleClass.ResultSoFar();
```

`foreach`ループでループ変数の型を決定するために暗黙の型指定を使用して良いです. 次の例では, foreachステートメントで暗黙の型指定が使用されています.

```cs
foreach (var ch in laugh)
{
    if (ch == 'h')
        Console.Write("H");
    else
        Console.Write(ch);
}
Console.WriteLine();
```

#### [補足]C#における`var`について

`var`は式の右辺の戻り値の型が自明な場合に使用できます. また, `var`はコンパイル時に型を決定します. したがって, 本格的な型推論に比べてコンパイル時間への影響は限定的であり, 実行時間への影響はありません. 

`var`が使える場面では積極的に`var`を使います. C#の言語仕様では暗黙的な型変換のオーバーロードが認められているので, 明示的な型で変数を定義するよりも`var`で定義したほうが安全になります.

#### Visual Studio で`var`を推奨する

`.editorconfig`に以下を追記します.

```
[*.cs]
#### C# Coding Conventions ####
# var preferences
csharp_style_var_elsewhere = true:suggestion
csharp_style_var_for_built_in_types = true:suggestion
csharp_style_var_when_type_is_apparent = true:suggestion
```

## C#のコーディング規約 への追加

### publicなメソッドの戻り値がValueTupleの時, 要素名はパスカルケースを使用します

`public`なメソッドの戻り値が`ValueTuple`の時, 要素名はパスカルケースを使用します. `private`の場合は規約はありません.

```cs
(var employeeNumber, var employeeName) = FetchEmployee(); // ここはprivateなので規約の対象外となる.
Console.Writeline(employeeNumber);
Console.Writeline(employeeName);

class C
{
    public (int EmployeeNumber, string EmployeeName) FetchEmployee() // publicメソッドなのでパスカルケースを使用する.
    {
        return (1, "浅井敏之");
    }
}
```

### DateTime型を使用しないようにします

DateTime型は全体で64ビット(62ビットのTicksと2ビットのKind)の構造体です. DateTimeが時刻を示すのは, KindがUTCの時に限ります. KindがLocalまたはUnspecifiedの時は, あいまいな時刻になります. 例えば`DateTime(2020, 1, 1, 0, 0, 0)`の時, 日本とアメリカでは異なる時刻を示します.
一方で, DateTimeOffsetは全体で80ビット(64ビットのDateTimeと16ビットのオフセット情報)の構造体です. オフセット=タイムゾーンの情報を持っているので, DateTimeOffsetは常に時刻を示します.
現代的にはUTC以外の時刻を扱う必要がないのでDateTime/UTCで良さそうなのですが, タイムゾーンが+の時は, 次のコードが通らずに困ります.

``` cs
DateTimeOffset dt0 = new DateTime(); // DateTimeOffsetに変換した時に日付が負の値になる. よって例外
DateTimeOffset dt1 = default(DateTime); // これは上のコードと等価. よって例外
```

ビジネスアプリケーション領域では, スペック要求が厳しいときもほとんどないので, DateTimeOffset使うと良いでしょう.

### List<T>型はIListインターフェースで受け取らないようにします.

C#ではList<T>型は代替のない型なので, 殆どの場合インターフェースを使って変数に代入する必要はありません. もしList<T>をインターフェースで受けたい場合はIEnumerable<T>を使用します.

IListインターフェース/IList<T>インターフェースはフレームワークに要求されない限り, 使用しないようにします. IListインターフェース/IList<T>インターフェースは, List<T>を表示するGUIで使用されます.


### 改行コードをcrlfにします

C#にはヒアドキュメント(改行や記号を含む文字列)があるので, 改行は有効なコードである可能性があります. `.gitattributes`ファイルと`.editorconfig`ファイルを記述して, 改行コードを指定します.

`.gitattributes`ファイル

```
*.cs text eol=crlf diff=csharp
*.tt text eol=crlf diff=csharp
```

`.editorconfig`ファイル

```
[*.{cs,tt}]
end_of_line = crlf
```

### 文字コードをBOM付きUTF8にします

C#はBOMなしのファイルはOSの既定に従って文字コードを認識します. 日本語環境のWindowsの既定の文字コードは`Shift-jis`ですが, docker環境などではUTF-8になるので, 特定の環境で特定の文字を含むファイルがビルドができなくなります. どの環境でもビルドできるようにBOM付きUTF8を指定します.

`.editorconfig`ファイル

```
[*.{cs,tt}]
charset = utf-8-bom
```

## その他の設定

### .gitignoreの設定

`.gitignore`ファイルを記述して, リポジトリに追跡しないファイルを選択します. Visual StudioやVisual Studio Codeが生成するファイルを把握するのは難しいので, `dotnet`コマンドを使用して`.gitignore`ファイルを作成します.

``` sh
dotnet new gitignore
```
