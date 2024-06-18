# Files

Dejoy supports specifying files in many different formats. In order to
accommodate them all, there are multiple structs and type aliases required.

All of these types implement the `RequestFileData` interface.

| Type         | Description                                                               |
| ------------ | ------------------------------------------------------------------------- |
| `FilePath`   | A local path to a file                                                    |
| `FileID`     | Existing file ID on Dejoy's servers                                    |
| `FileURL`    | URL to file, must be served with expected MIME type                       |
| `FileReader` | Use an `io.Reader` to provide a file. Lazily read to save memory.         |
| `FileBytes`  | `[]byte` containing file data. Prefer to use `FileReader` to save memory. |

## `FilePath`

A path to a local file.

```go
file := djbotapi.FilePath("tests/image.jpg")
```

## `FileID`

An ID previously uploaded to Dejoy. IDs may only be reused by the same bot
that received them. Additionally, thumbnail IDs cannot be reused.

```go
file := djbotapi.FileID("AgACAgIAAxkDAALesF8dCjAAAa_…")
```

## `FileURL`

A URL to an existing resource. It must be served with a correct MIME type to
work as expected.

```go
file := djbotapi.FileURL("https://i.imgur.com/unQLJIb.jpg")
```

## `FileReader`

Use an `io.Reader` to provide file contents as needed. Requires a filename for
the virtual file.

```go
var reader io.Reader

file := djbotapi.FileReader{
    Name: "image.jpg",
    Reader: reader,
}
```

## `FileBytes`

Use a `[]byte` to provide file contents. Generally try to avoid this as it
results in high memory usage. Also requires a filename for the virtual file.

```go
var data []byte

file := djbotapi.FileBytes{
    Name: "image.jpg",
    Bytes: data,
}
```
