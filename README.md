# NCU-linux-project1
linux作業系統 project1
## 加入system call 並 編譯OS  
參考 https://blog.kaibro.tw/2016/11/07/Linux-Kernel%E7%B7%A8%E8%AD%AF-Ubuntu/  
## project1  
**證明每個process 的kernel space 都是指到同樣的physical address**  
**也就是每個process 的kernel space 是共用的**  

do_fork=>copy_process=>copy_mm=>dup_mm=>mm_init=>mm_alloc_pgd=>pgd_alloc=>pgd_ctor=>clone_page_range  

![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202021-02-17%20110324.png)  

![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202021-02-17%20110425.png)  

![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202021-02-17%20110502.png)  

![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202021-02-17%20110554.png)  

![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202021-02-17%20110631.png)  

![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202021-02-17%20110706.png)  

![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202021-02-17%20110738.png)  

![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/%E8%9E%A2%E5%B9%95%E6%93%B7%E5%8F%96%E7%95%AB%E9%9D%A2%202021-02-17%20110823.png)  

經過trace code後可以發現每當process做fork的時候  
就會複製kernel space的位置給新的process  
所以每個process的kernel space 都會指向同一個physical address  

## 證明  
### 作法
用for_each_process這個macro 去遍歷每個process  
然後在process descriptor 中的 mm_struct可以找到 pgd的位置  
再利用pgd => pud => pmd =>pte 找到physical address  
所以我們只要去看我們丟進去的virtual address 在kernel space的時候(也就是>0xc0000000) 每個process轉換出來的physical address 是否一樣就可以了  

## System code  
```C
#include<linux/init.h>
#include<linux/pid.h>
#include<linux/sched.h>
#include<linux/module.h>
#include<linux/mm_types.h>
#include<linux/kernel.h>
#include<asm/pgtable.h>
#include<linux/highmem.h>


asmlinkage void sys_solve2(unsigned long addr){
  struct task_struct* task=NULL;
  int count=0;
  for_each_process(task){
    count++;
    printk("count%d:\n",count);
    printk("pid=%d\n",task->pid);
    pgd_t *pgd;
    pte_t *ptep;
    pte_t pte;
    pud_t *pud;
    pmd_t *pmd;
    pmd_t pmd2;
    unsigned long pa=0;
    unsigned long page_addr=0;
    unsigned long page_offset=0;
    struct mm_struct* mm=NULL;
    struct page *page=NULL;
    mm=get_task_mm(task);
    if(mm){
      pgd = pgd_offset(mm, addr);
      pud = pud_offset(pgd, addr);
      pmd = pmd_offset(pud, addr);
      ptep = pte_offset_map(pmd, addr);
      pte = *ptep;
      pmd2= *pmd;
      if(addr>=0xc0400000){
        page_addr=pmd_val(*pmd) & PAGE_MASK;
        page_offset=addr & ~PAGE_MASK;
        pa=page_addr | page_offset;
        printk("pmd mask: @ %08x\n",pa);
      }
      page_addr=pte_val(*ptep) & PAGE_MASK;
      page_offset=addr & ~PAGE_MASK;
      pa=page_addr | page_offset;
      printk("mask: @ %08x\n",pa);
    }
    else{
      printk("mm=NULL\n");
    }
  }
}
```  
**因為0xc0400000 後是用4M mapping 所以只轉換到pmd**
![image](https://github.com/fallantbell/NCU-linux-project1/blob/main/aaa.png)  
