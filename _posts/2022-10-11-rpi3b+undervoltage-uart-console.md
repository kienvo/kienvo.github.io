---
layout: post
title:  "Lỗi console uart trên rpi3b+"
image: '/assets/rpi3b+-undervoltage-uart-console/console.png'
date: 2022-10-11 12:00 +0700
tags: [raspberry-pi, rpi3b+, console, uart, under-voltage, linux]
description: 'Sửa lỗi nhảy kí tự trên console uart, Raspberry Pi 3B+'
categories: [raspberry-pi]
---

## Lỗi

Khi sử dụng rpi3b+ với nguồn usb laptop hoặc mấy loại củ sạc dỏm thì console uart của rpi bị nhảy kí tự, không thể đánh lệnh hoặc không thể đọc được.

## Nguyên nhân

Mấy củ sạc dỏm hay nguồn usb laptop không đủ dòng (hoặc áp) cho rpi. Chắc là do
cố ý thiết kế, chứ 5v đưa vào cpu thì vẫn phải hạ xuống 1.8v và 0.9v. Có thể đây là 1 `tính năng` để người dùng mua hàng từ Raspberry Pi Foundation.

## Cách khắc phục

Sử dụng nguồn xịn, lọc nhiễu kĩ. Như tôi thử với củ sạc Xiaomi chạy ngon lành, cần đeo' gì nguồn chính hãng Raspberry Pi.

Cách thứ 2, khi không muốn hoặc không có nguồn xịn, đó là giảm `core_freq` từ 300 xuống 250. Chỉ cần thêm dòng `core_freq=250` vào cuối file config.txt hoặc sửa `core_freq=300` thành `core_freq=250`. Mỗi distro linux cho arm sẽ có file config.txt ở 1 nơi khác nhau, nhưng thông thường là ở `/boot`. Như tôi đang xài ubuntu bionic cho rpi3b+, file config có đường dẫn `/boot/firmware/config.txt` nhưng file config này có include 2 file khác là `/boot/firmware/usercfg.txt` và `/boot/firmware/syscfg.txt` nên tôi thêm `core_freq=250` vào file usercfg.txt.

![usercfg.txt](/assets/rpi3b+-undervoltage-uart-console/usercfg.txt.png)
