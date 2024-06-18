---
title: "Xử lý data race trong Golang"
date: "2024-06-09"
# description: "Hướng dẫn thu thập dữ liệu chứng khoán từ công ty X cùng Python và Google Sheet"
tags: ["golang", "go", "data-race"]
categories: ["Golang thật là đơn giản"]
series: ["Golang thật là đơn giản"]
cover:
  image: data-race.png
  caption: Lazy gopher image by [github.com/ashleymcnamara/gophers](github.com/ashleymcnamara/gophers)
ShowToc: true
TocOpen: true
---

## Data race là gì?

- **Data race** là hiện tượng xảy ra khi có từ 2 *goroutines* cùng truy cập (*read* hoặc *write*) vào vùng nhớ chung (*shared resource*).
- Có ít nhất một hành động thay đổi dữ liệu ở vùng nhớ chung (phương thức *write*).
- Đối với các ứng dụng chạy đơn luồng (*single thread*), **data race** sẽ không xảy ra. 

---

## Các trường hợp xảy ra data race

### Trùng biến index vòng lặp
```go {linenos=table,hl_lines=[2],linenostart=1}
nums := []int{19, 12}
for _, n := range nums {
  go func() {
    fmt.Printf("address: %p, value: %d\n", &n, n)
  }()
}
```
- Vòng lặp thứ nhất:
  - Biến `n` với địa chỉ 0x0001 được khởi tạo.
  - Biến `n` được gán giá trị của phần tử đầu tiên (`n` = 19, `&n` = 0x0001).
  - Đưa biến `n` vào *goroutine*
- Vòng lặp thứ hai:
  - Biến `n` được gán giá trị của phần tử thứ hai (`n` = 12, `&n` = 0x0001).
  - Đưa biến `n` vào *goroutine*

Trong trường hợp này *goroutine* thứ nhất thực thi sau khi biến `n` gán giá trị của phần tử thứ hai khiến **data race** xảy ra.

*\*Lưu ý: Từ Go version 1.22 trở đi [vòng **for**](https://tip.golang.org/doc/go1.22#language) đã được điều chỉnh lại, giờ đây mỗi vòng lặp sẽ khởi tạo lại biến `n` giúp tránh được **data race** trong trường hợp trên.\**

### Trùng biến error

```go {linenos=table,hl_lines=[11,13],linenostart=1}
func Foo() error {
	return fmt.Errorf("error in Foo")
}

func Bar() error {
	return fmt.Errorf("error in Bar")
}

func main() {
  // ...
	err := Foo()
	go func() {
		err = Bar()
	}()

  // ...
}
```
- Trong trường hợp này, vì biến `err` không được khởi tạo lại trong hàm *goroutine* (dòng 13) nên biến `err` (dòng 10) sẽ bị cập nhật lại một cách không mong muốn dẫn đến **data race** xảy ra.

### Data race trong map

- `map` trong Go [không hỗ trợ xử lý đồng thời](https://go.dev/blog/maps#concurrency)
- Khi các goroutines xử lý đồng thời (cả read và write) sẽ dẫn đến data race.

```go {linenos=table,hl_lines=[8],linenostart=1}
nums := []int{19, 12}
var unsafeMap = make(map[int]int)
var wg sync.WaitGroup
wg.Add(len(nums))
for _, n := range nums {
  go func(key int) {
    defer wg.Done()
    unsafeMap[key] = key
  }(n)
}
wg.Wait()
```
Tại dòng 8, nếu có từ 2 *goroutines* xử lý cùng lúc sẽ dẫn đến **data race**. Trong ví dụ trên, cần phải chạy nhiều lần hoặc là dữ liệu đủ nhiều chúng ta mới có thể quan sát được **data race** xảy ra.

### Data race trong slice

*Lưu ý: Cần có kiến thức về [slice](https://go.dev/blog/slices-intro) trước khi đọc phần này*

```go {linenos=table,hl_lines=[2,8],linenostart=1}
nums := []int{19, 12}
nums = append(nums, 1)
var wg sync.WaitGroup

wg.Add(1)
go func(numsLocal []int) {
  defer wg.Done()
  numsLocal[0] = -9999
}(nums)
wg.Wait()
fmt.Println(nums)
```
- Dòng 2, mở rộng `nums` capacity.
- Dòng 9, truyền `nums` như tham trị vào hàm *goroutine*.
- Dòng 8, **data race** xuất hiện.

*(đang cập nhật...)*

---

## Tài liệu tham khảo
- [A Study of Real-World Data Races in Golang](https://arxiv.org/pdf/2204.00764)