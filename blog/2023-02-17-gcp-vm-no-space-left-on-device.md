# 突然連不進 GCP 虛擬機器的處理記錄

## 事發經過

今天凌晨把程式碼推上 bitbucket，pipeline 顯示發生錯誤

```
file_put_contents(): write of 143 bytes failed with errno=28 No space left on device
```

pipeline 的設定是 ssh 連進虛擬機器然後更新程式碼，錯誤訊息是機器已經沒有空間了，當初選的硬碟容量是 10G，我也沒有在注意使用量，但確實會持續新增使用者上傳的圖片，我想用 ssh 連進去檢查但連不進去，GCP 的後台也沒有看到硬碟剩餘空間相關的資訊，但機器是開著的狀態，先建立快照(之前也沒有建立)，重開機器，結果也是一樣，我想說會不會先增加容量再重開機就有機會讓我 ssh 進去，於是我開始搜尋要怎麼處理，找到 GCP 的 [troubleshooting-disk-full-resize](https://cloud.google.com/compute/docs/troubleshooting/troubleshooting-disk-full-resize)，其中描述的情境跟我遇到的很像，於是我照著步驟

1. 關機重開(已經做過)
2. 關機建立 snapshot(已經做過)
3. 升級成 20G

```
$ gcloud compute instances stop VM_NAME

$ gcloud compute disks resize BOOT_DISK_NAME --size DISK_SIZE
// 這一步還需要填 region 或是 zone，GCP 後台可以查，size 我是填 20G

$ gcloud compute instances start VM_NAME
```

4. 重開可以 ssh 進去了(真是太好了)，如果還是不行，文件中也有提供兩種做法，但我沒去嘗試了
5. df -h 看容量不太對，跟著的指示，設定[這一篇](https://cloud.google.com/compute/docs/disks/resize-persistent-disk)好就正常了

```
$ df -Th

$ lsblk

$ parted -l
or
$ parted -l /dev/DEVICE_NAME

$ parted /dev/BOOT_DISK_NAME
# 需要輸入 resizepart
# 需要輸入要分區號碼，這次我輸入 1
# 出現警告提醒這個分區使用中
# end? 這次我輸出 100%
# 輸入 quit

$ partprobe /dev/DEVICE_NAME
# 讀取新的分區表

# 擴展 file system (要先看分區的類型是 ext4 還是 xfs，ext4 可以用以下命令擴展)
$ resize2fs /dev/PARTITION_NAME
```

這些操作完成後並不需要重啟 instance，可以用 df 命令確認是否成功

## 心得

突然遇到服務都停掉、機器也進不去有點緊張，幸好增加容量之後就順利解決問題，當初在開 VM 的時候選擇 10G 可能太小了，
VM 重啟後對外 IP 會變，因此 DNS 跟服務中有設定到 IP 的地方都要記得調整，後續打算設定成靜態 IP 以減少不必要的錯誤，
下一步可以做的事情就是備份跟監控，整個虛擬機可以做快照，萬一硬碟真的壞掉的時候還可以用快照重開一台，
如果有一個機制可以定期的檢查硬碟可用空間可以適時提醒管理人員進行調整，把使用者上傳的檔案另外存到像是 S3 這種服務也是另一種策略
