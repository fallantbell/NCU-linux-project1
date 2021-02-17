# NCU-linux-project1
linux作業系統 project1
## 加入system call 並 編譯OS  
參考 https://blog.kaibro.tw/2016/11/07/Linux-Kernel%E7%B7%A8%E8%AD%AF-Ubuntu/  
## project1  
**證明每個process virtual address 的kernel space 都是指到同樣的physical address**  
**也就是每個process 的kernel space 是共用的**  

do_fork=>copy_process=>copy_mm=>dup_mm=>mm_init=>mm_alloc_pgd=>pgd_alloc=>pgd_ctor=>clone_page_range  

