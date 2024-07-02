---
title: "Tối ưu hiệu suất cho Go"
date: "2024-07-01"
tags: ["golang", "go"]
categories: ["Golang thật là đơn giản"]
series: ["Golang thật là đơn giản"]
cover:
  image: MovingGopher.png
  caption: Moving gopher image by [github.com/ashleymcnamara/gophers](github.com/ashleymcnamara/gophers)
ShowToc: true
TocOpen: true
---

## Dạng tham trị và các biến có kiểu kích thước lớn

- Hạn chế sử dụng dạng tham trị đối với các biến có kiểu kích thước lớn để tối ưu tốc độ.
- Kiểu kích thước lớn là các `struct` hoặc `array` có kiểu kích thước lớn hơn 4 word (với kiến trúc 32-bit, 1 word = 4 bytes, với kiến trúc 64-bit, 1 word = 8 bytes).
- Đối với kiểu kích thước nhỏ, compiler Golang đã tối ưu hoá. 

```go
// kiểu array có kích thước lớn
type X [5]int

// kiểu struct có kích thước lớn
type Y struct {
  a, b, c, d, e int
}
```

### Ví dụ tính tổng của X
```go {linenos=table,hl_lines=[2,7,15],linenostart=1}
type X struct {
  a, b, c, d, e int
}

func Sum_OneVariable(numbers []X) (int) {
  var r int
  for i := range s {
    r += s[i].a
  }
  return r
}

func Sum_TwoVariable(numbers []X) (int) {
  var r int
  for _, v := range s {
    r += v.a
  }
  return r
}


func createSliceX() []X {
	var s = make([]X, 1000)
	for i := range s {
		s[i] = X{a: i}
	}
	return s
}

var result [128]int

func Benchmark_OneVariable(b *testing.B) {
	var s = createSliceX()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		result[i&127] = Sum_OneVariable(s)
	}
}

func Benchmark_TwoVariable(b *testing.B) {
	var s = createSliceX()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		result[i&127] = Sum_TwoVariable(s)
	}
}

```

- Trong trường hợp trên, khi sử dụng `for` với hai biến, biến thứ hai sẽ được gán dưới dạng tham trị (dòng 15).
- Nếu bỏ bớt một thuộc tính trong `X` (dòng 2) và chạy lại *benchmark* hiệu suất sẽ có sự thay đổi.
- Một vài trường hợp khác:
  - Gán giá trị vào `map`.
  - `append` phần tử vào trong `slice`.
  - (đang cập nhật)

---

## Cấp phát bộ nhớ vừa đủ khi dùng slice

- Khi dùng `slice`, ta nên tính toán kích thước bộ nhớ cần dùng và cấp phát một lần. 

### Ví dụ cấp phát bộ nhớ khi merge mảng

```go {linenos=table,hl_lines=[2],linenostart=1}
func getData() [][]int {
	return [][]int{
		{1, 2},
		{9, 10, 11},
	}
}

func MergeWithOneLoop(data ...[]int) []int {s
	var r []int
	for _, d := range data {
		r = append(r, d...)
	}
	return r
}

func MergeWithTwoLoops(data ...[]int) []int {
	n := 0
	for _, d := range data {
		n += len(d)
	}
	r := make([]int, 0, n)
	for _, d := range data {
		r = append(r, d...)
	}
	return r
}

var result [128]int

func Benchmark_MergeWithOneLoop(b *testing.B) {
	data := getData()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		result[i&127] = MergeWithOneLoop(data...)
	}
}

func Benchmark_MergeWithTwoLoops(b *testing.B) {
	data := getData()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		result[i&127] = MergeWithTwoLoops(data...)
	}
}
```

- Với trường hợp 1 vòng lặp, quá trình cấp phát sẽ diễn ra mỗi khi `cap` của `slice` đạt đến ngưỡng tối đa.
- Với trường hợp 2 vòng lặp, quá trình cấp phát chỉ xảy ra 1 lần (dòng 21).

---

## Tài liệu tham khảo
- [Go Optimizations 101](https://go101.org/optimizations/101.html)