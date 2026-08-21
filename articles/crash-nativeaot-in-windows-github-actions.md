---
title: .NET の NativeAOT な実行ファイルが GitHub Actions の Windows 環境でクラッシュする
emoji: "😵‍💫"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - GitHubActions
  - PowerShell
published: true
---

# .NET の NativeAOT な実行ファイルのクラッシュ

2026年8月現在、GitHub Actions の Windows 環境では .NET の NativeAOT 実行ファイルがクラッシュする可能性があります。

以下の場合にクラッシュすることがあります。

- ビルドするジョブと実行するジョブが別
  - ビルドジョブで upload-artifact、実行ジョブで download-artifact のようにするパターンです。
- `dotnet publish -c Release -p:PublishAot=true -p:IlcInstructionSet=native` のように IlcInstructionSet=native としている
  -  .csproj ファイルに `<IlcInstructionSet>native</IlcInstructionSet>` とした場合も同様

> The required instruction sets are not supported by the current CPU.

というエラーでクラッシュする場合があります。

:::details クラッシュを引き起こすワークフロー

```yaml
name: Windows env

on:
  workflow_dispatch:
jobs:
  build-aot:
    runs-on: windows-latest
    strategy:
      matrix:
        # prettier-ignore
        index:
          ["00", "01", "02", "03", "04", "05", "06", "07", "08", "09"]
    steps:
      - uses: actions/setup-dotnet@v6
        with:
          dotnet-version: '10.0.x'
      - name: build
        run: |
          echo 'System.Console.WriteLine("hello!");' > hello${{matrix.index}}.cs
          dotnet publish .\hello${{matrix.index}}.cs -c Release -p:PublishAot=true -p:IlcInstructionSet=native -o publish
      - uses: actions/upload-artifact@v7
        with:
          name: hello-${{matrix.index}}
          path: ${{ github.workspace }}/publish/hello${{matrix.index}}.exe
          retention-days: 1
  run-aot:
    needs: [build-aot]
    runs-on: windows-latest
    strategy:
      matrix:
        # prettier-ignore
        index:
          ["00", "01", "02", "03", "04", "05", "06", "07", "08", "09"]
    steps:
      - uses: actions/download-artifact@v7
        with:
          pattern: hello-*
      - shell: pwsh
        run: |
          dir */hello*.exe | %{
           echo "Run: $_"
           . $_
          }

```

:::


# Hardware Intrinsicsの調査

Hardware Intrinsics に対応しているかどうかは PowerShell で調べられるのでワークフローを並列実行して調べてみましょう。


:::details Hardware Intrinsics を調べるワークフロー

```yaml
name: Windows env

on:
  workflow_dispatch:
jobs:
  intrinsics:
    runs-on: windows-latest
    strategy:
      matrix:
        # prettier-ignore
        index:
          ["00", "01", "02", "03", "04", "05", "06", "07", "08", "09"]
    steps:
      - name: Intrinsics
        shell: pwsh
        run: |
          @{
            "X86Base"= $([System.Runtime.Intrinsics.X86.X86Base]::IsSupported);
            "Aes"= $([System.Runtime.Intrinsics.X86.Aes]::IsSupported);
            "Avx"= $([System.Runtime.Intrinsics.X86.Avx]::IsSupported);
            "Avx2"= $([System.Runtime.Intrinsics.X86.Avx2]::IsSupported);
            "Avx10v1"= $([System.Runtime.Intrinsics.X86.Avx10v1]::IsSupported);
            "Avx10v2"= $([System.Runtime.Intrinsics.X86.Avx10v2]::IsSupported);
            "AvxVnniInt8"= $([System.Runtime.Intrinsics.X86.AvxVnniInt8]::IsSupported);
            "AvxVnniInt16"= $([System.Runtime.Intrinsics.X86.AvxVnniInt16]::IsSupported);
            "Avx512BW"= $([System.Runtime.Intrinsics.X86.Avx512BW]::IsSupported);
            "Avx512CD"= $([System.Runtime.Intrinsics.X86.Avx512CD]::IsSupported);
            "Avx512DQ"= $([System.Runtime.Intrinsics.X86.Avx512DQ]::IsSupported);
            "Avx512F"= $([System.Runtime.Intrinsics.X86.Avx512F]::IsSupported);
            "Avx512Vbmi"= $([System.Runtime.Intrinsics.X86.Avx512Vbmi]::IsSupported);
            "Avx512Vbmi2"= $([System.Runtime.Intrinsics.X86.Avx512Vbmi2]::IsSupported);
            "AvxVnni"= $([System.Runtime.Intrinsics.X86.AvxVnni]::IsSupported);
            "Bmi1"= $([System.Runtime.Intrinsics.X86.Bmi1]::IsSupported);
            "Bmi2"= $([System.Runtime.Intrinsics.X86.Bmi2]::IsSupported);
            "Fma"= $([System.Runtime.Intrinsics.X86.Fma]::IsSupported);
            "Gfni"= $([System.Runtime.Intrinsics.X86.Gfni]::IsSupported);
            "Lzcnt"= $([System.Runtime.Intrinsics.X86.Lzcnt]::IsSupported);
            "Pclmulqdq"= $([System.Runtime.Intrinsics.X86.Pclmulqdq]::IsSupported);
            "Popcnt"= $([System.Runtime.Intrinsics.X86.Popcnt]::IsSupported);
            "Sse"= $([System.Runtime.Intrinsics.X86.Sse]::IsSupported);
            "Sse2"= $([System.Runtime.Intrinsics.X86.Sse2]::IsSupported);
            "Sse3"= $([System.Runtime.Intrinsics.X86.Sse3]::IsSupported);
            "Sse41"= $([System.Runtime.Intrinsics.X86.Sse41]::IsSupported);
            "Sse42"= $([System.Runtime.Intrinsics.X86.Sse42]::IsSupported);
            "Ssse3"= $([System.Runtime.Intrinsics.X86.Ssse3]::IsSupported);
            "X86Serialize"= $([System.Runtime.Intrinsics.X86.X86Serialize]::IsSupported);
          } | ConvertTo-Json > ${{ github.workspace }}/result${{matrix.index}}.json

      - uses: actions/upload-artifact@v7
        with:
          name: outputs-${{ matrix.index }}
          path: ${{ github.workspace }}/result${{matrix.index}}.json
          retention-days: 1
  merge:
    needs: [intrinsics]
    runs-on: ubuntu-slim
    steps:
      - uses: actions/download-artifact@v7
        with:
          path: outputs

      - name: merge
        shell: pwsh
        run: |
          $files = Get-ChildItem "outputs/*/result*.json" | Sort-Object Name
          echo $files

          $dataList = @()
          $headers = @("name")
          for ($i = 0; $i -lt $files.Count; $i++) {
              $json = Get-Content $files[$i].FullName -Raw | ConvertFrom-Json
              $dataList += [PSCustomObject]@{
                  Index   = $i + 1
                  Content = $json
              }
              $headers += ($i + 1)
          }

          $keys = $dataList[0].Content.PSObject.Properties.Name

          ($headers -join "|") > $env:GITHUB_STEP_SUMMARY
          (($headers | % { if ($_ -eq "name") { ":---" } else { "---" } }) -join "|") >> $env:GITHUB_STEP_SUMMARY

          foreach ($key in $keys) {
              $row = @($key)
              foreach ($data in $dataList) {
                  $val = $data.Content.$key
                  if ($val -is [bool]) { $val = ($val ? 1 : 0) }
                  $row += $val
              }
              ($row -join "|") >> $env:GITHUB_STEP_SUMMARY
          }
```

:::

実行すると、X86Serialize や AvxVnni などが一部の環境でだけ有効になっています。有効なCPU命令が多い環境でビルドした.exeファイルを少ない環境で実行しようとするとクラッシュしてしまうのです。

name|1|2|3|4|5|6|7|8|9|10
:---|---|---|---|---|---|---|---|---|---|---
Aes|1|1|1|1|1|1|1|1|1|1
AvxVnniInt16|0|0|0|0|0|0|0|0|0|0
Lzcnt|1|1|1|1|1|1|1|1|1|1
X86Base|1|1|1|1|1|1|1|1|1|1
X86Serialize|0|0|1|0|0|0|0|0|0|0
Avx512DQ|0|0|1|0|0|0|0|0|0|0
Avx512F|0|0|1|0|0|0|0|0|0|0
Pclmulqdq|1|1|1|1|1|1|1|1|1|1
Sse|1|1|1|1|1|1|1|1|1|1
Bmi2|1|1|1|1|1|1|1|1|1|1
Avx512BW|0|0|1|0|0|0|0|0|0|0
Popcnt|1|1|1|1|1|1|1|1|1|1
Sse42|1|1|1|1|1|1|1|1|1|1
Avx2|1|1|1|1|1|1|1|1|1|1
Avx|1|1|1|1|1|1|1|1|1|1
Sse2|1|1|1|1|1|1|1|1|1|1
Ssse3|1|1|1|1|1|1|1|1|1|1
Bmi1|1|1|1|1|1|1|1|1|1|1
AvxVnni|0|0|1|0|0|0|0|0|0|0
Sse3|1|1|1|1|1|1|1|1|1|1
Avx512CD|0|0|1|0|0|0|0|0|0|0
AvxVnniInt8|0|0|0|0|0|0|0|0|0|0
Gfni|0|0|1|0|0|0|0|0|0|0
Avx10v2|0|0|0|0|0|0|0|0|0|0
Sse41|1|1|1|1|1|1|1|1|1|1
Avx10v1|0|0|0|0|0|0|0|0|0|0
Fma|1|1|1|1|1|1|1|1|1|1
Avx512Vbmi2|0|0|1|0|0|0|0|0|0|0
Avx512Vbmi|0|0|1|0|0|0|0|0|0|0

# 対処方法

`<IlcInstructionSet>native</IlcInstructionSet>` を使わず、必要な CPU 命令を `<IlcInstructionSet>avx2</IlcInstructionSet>` のように直接指定してあげましょう。

