---
title: "Xử lý data race trong Golang"
date: "2024-06-09"
# description: "Hướng dẫn thu thập dữ liệu chứng khoán từ công ty X cùng Python và Google Sheet"
tags: ["golang"]
categories: ["Golang thật là đơn giản"]
series: ["Golang thật là đơn giản"]
# cover:
#   image: docker-logo.png
#   caption: ""
ShowToc: true
TocOpen: true
---

## Data race là gì?

- Data race là hiện tượng xảy ra khi có từ 2 goroutines cùng truy cập (*read* hoặc *write*) vào vùng nhớ chung (*shared resource*).
- Có ít nhất 1 goroutine thực hiện việc thay đổi dữ liệu (phương thức *write*).
- Đối với các ứng dụng chạy đơn luồng (*single thread*), data race sẽ không xảy ra. 

---

## Các trường hợp xảy ra data race

### Trùng biến phần tử trong vòng lặp

```
func ProcessJob(job *Job) {
  ...
}

for _ , job := range jobs {
  go func () {
    ProcessJob(job) // 
  }()
}
```

Khi hàm `ProcessJob` được đưa vào goroutine để xử lý vòng lặp thứ nhất thì biến `job` đang trỏ đến phần tử đầu tiên. Đến vòng lặp thứ hai, biến `job` sẽ trỏ đến phần tử kế tiếp của mảng và làm sai lệch dữ liệu đầu vào của goroutine đầu. Điều này dẫn đến data race.

---



## Mở rộng
Từ đây, chúng ta có thể thêm tuỳ ý các service khác (database, message queue,...) vào trong `docker-compose.yaml`.