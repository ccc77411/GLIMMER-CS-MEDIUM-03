# MEDIUM-3：初识Linux系统  
## 第一境：算者—— 命令行参数与路径  
### 【任务 1.1：不同的路】  
#### 1.请解释什么是绝对路径，什么是相对路径？  
- **绝对路径**：是以根目录为起点来描述一个路径的方法，路径以/（即根目录）开头。  
- **相对路径**：是以当前目录为起点来描述一个路径的方法，路径不需以/开头。  
#### 假设你现在正处于 /home/lin/workspace/，请分别写出前往 /var/backups/turing/ 的绝对路径命令和相对路径命令（假设 cd 命令还能用）。  
- **绝对路径**：cd /var/backups/turing/  
- **相对路径**: cd ../../../var/backups/turing/  
### 【任务 1.2：何处启程】  
#### 1.简要解释主函数参数 argc 与 argv[] 的含义是什么。  
- 参数argc含义是参数计数，表示表示命令行中参数的个数（参数包括程序名本身）   
- 参数argv[]含义是参数向量，表示每个参数的具体内容。  
#### 2.针对上述命令，请问 argc 的值是多少？argv[0]、argv[1]、argv[2] 分别指向什么字符串？  
- 针对上述cd命令，argc（即参数个数）的值是2，因为包含了程序名“cd”本身和路径。 
- argv[0]指向“cd”，argv[1]指向用户传入的路径“/var/backups/turing/”或“../../../var/backups/turing/”，argv[2]在规定中为NULL，即不指向任何参数字符串为空指针，只是作为数组结束的标志。   
## 第二境：算王—— 重建 ls 命令与位运算  
### 【任务 2.1：非同寻常之物】  
#### 1.以上哪些头文件是你见过的，哪些是你没见过或者很少见到的？  
- 见过的头文件有`<stdio.h>`（最常见）`<string.h>`（在之前查资料时见过）`<stdlib.h>`（分配动态内存的时候用过）   
- `<dirent.h>` `<sys/stat.h>``<sys/types.h>`` <time.h>`这些都没见过。   
#### 2.`<dirent.h>` 和` <sys/stat.h>` 分别提供了哪些关键的结构体或函数？它们分别有什么作用?（简要回答即可，不用详细列举所有作用）    
- **dirent.h（主要用于列出目录）**：  
1. 提供了`struct dirent`这个关键结构体，代表了代表一个目录项，其中 d_name 字段就是文件/子目录的名字。 
2. 提供了`opendir(path)`这一个函数，用于打开一个目录。    
3. 提供了`readdir()`函数，用于逐个读出目录项，直至返回NULL。    
4. 提供了` closedir()`函数，用于关闭目录。  
- **sys/stat.h（主要用于表示文件信息）**    
1. 提供了`struct stat`这个关键结构体，用于存储文件信息。  
2. 提供了`stat(path, &buf)`函数，用于按路径取属性。只有`readdir()`拿到名字，`stat()`才能根据名字拼出路径，拿到属性。       
### 【任务 2.2：看看权限】  
#### 请思考：整数如何表示文件的权限    
- 应该是利用整数的二进制表示表示的，规定某一位对应哪个权限，那么该位数为1，则表示有这一个权限，若为0，则表示没有这一个权限。   
#### 为了让重写的myls具备显示权限的功能，请用最少的代码和位运算，补全下方 C 语言函数，将整数形式的权限位转换为 9个字符的权限字符串。   
- **补全的函数**  
````  
// mode即传入的存储权限信息的整数， 如0754（八进制表示）
// 提示：传入的 mode 变量不仅包含权限信息，高位可能还包含文件类型。在进行位运算时，请注意屏蔽（Mask）掉不需要的高位数据。
void parse_permissions(mode_t mode, char *perms) {

    // 权限字符对照表
    char rwx[] = {'r', 'w', 'x'};  

    // 请补全代码写在 TO DO 与 END OF TO DO 之间：
    // TO DO
    mode&=511;                                         //屏蔽掉不需要的高位数据
   for ( int i = 0; i < 9; i++)
   {
    perms[i]=(mode&256)?rwx[i%3]:'-';                  //利用二进制位的0，1表示，来看是否有此权限，并向perms数组中赋值。
    mode<<=1;                                          //让下一位移上来。
   }
    // END OF TO DO

    perms[9] = '\0'; // 字符串结尾
}
````  
## 第三境：算尊—— 深入理解 Inode 与文件树  
### 【任务 3.1：万物皆 Inode】  
#### 1.请简述 Inode 中存储了文件的哪些信息？  
- 存储了Inode编号，文件的类型和权限，所有者和所属组的ID，文件的大小，时间戳（包括访问时间，修改时间和状态改变时间），链接数指向该Inode的文件名数量，数据块指针（指向存储文件数据的数据块）。    
#### 2.Inode中不包含文件的什么信息？  
- 不包含文件的文件名和路径，因为这是存在父目录中的。          
#### 3.每一个目录下必定存在两个隐藏的特殊目录："." 和 ".."它们分别代表什么？它们的 Inode 编号在系统根目录"/"下有什么特殊规律？  
- “.”指向该目录自身的Inode，用来表示当前目录的名字。  
- “..”指向父目录的Inode，用来表示父目录名字，便于向上索引（如cd ..）。  
- 在系统根目录下这两个的编号完全相同，因为根目录已经是最上面的目录了，它没有父目录，所以让“..”目录也指向自身的Inode，所以这两个目录指向的Inode都是根目录本身的Inode，所以他们的Inode编号相同，都等于根目录自己的Inode编号。
### 【任务 3.2：寻我来时路】    
**mypwd.c的补全**  
````  
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/stat.h>
#include <dirent.h>

ino_t get_inode(const char *path) {
    struct stat statbuf;
    if (stat(path, &statbuf) == -1) {
        perror("stat failed");
        exit(EXIT_FAILURE);
    }
    return statbuf.st_ino;
}


void get_path(int *i,ino_t b[]){
    ino_t a;
    a=get_inode(".");                  //先获取当前目录的Inode编码
    chdir("..");        
    while(1){
    if (*i >= 100) { 
        fprintf(stderr, "层级过深\n");          //讨论层数过深的情况  
        exit(EXIT_FAILURE); 
    }
    b[*i]=a;                                    //这里一直向上记录路径中目录的Inode编码
    if(get_inode("..")==get_inode(".")){
       break;                                  //看是否为根节点
    }
    a=get_inode(".");
    chdir("..");                              
    (*i)++;
    }
} 


void print_path(int i,ino_t b[]){
     DIR *dir ;
    struct dirent *entry1;
    while (i>=0)
    {
       int found=0;                               
       if (!(dir = opendir("."))) {        //看是否正常打开目录获取句柄
        perror("opendir");
        exit(EXIT_FAILURE);
    }
    while ((entry1 = readdir(dir)) != NULL)         //找到编号对应的目录名字并打印，换到这一目录
    {
        if (entry1->d_ino==b[i])
        {
            printf("/%s",entry1->d_name);
            chdir(entry1->d_name);
            found=1;
            break;
        }
    }
    closedir(dir);
    if (!found)                                           //判断是否出现没找到目录的情况  
    {
        fprintf(stderr, "找不到 inode %lu\n", (unsigned long)b[i]);
        exit(EXIT_FAILURE);
    }
    i--;
    }
    printf("\n");
}


int main(){
    int i=0;
    ino_t b[100];
    if (get_inode("..")==get_inode("."))    //根节点的判特
    {
        printf("/\n");
        return 0;
    }
    
    get_path(&i,b);                    //调用之前定义的函数
    print_path(i,b);
    return 0;
}
````  
**思考过程**:  
- 刚看完这一题的时候是还没什么头绪的，然后我就去查了一下`chdir()``opendir()`和`readdir()`的用法，再结合前面对ls命令的实现，我就有了想法，一开始我想的是先读取一下当前目录的Inode编号，然后转到父目录，在父目录中找到与这一编号相同的子目录然后打印出来，这样一直向上，直到根目录。但写到一半发现这样打印出来的路径是反过来的。所以我就想到了用数组先保存路径中每个目录的Inode编号，到头之后，再根据编号来向下找到路径中的目录，打印出名字。依照这个大概的自己先写出来了一版，交给AI检查。果不其然一堆bug，然后依照AI提示写出了上面的代码。在这一题AI主要是帮我检查出bug，加上一些自己没想到的情况，大体部分是我自己写的。
    
**AI的prompt**  
<img width="1831" height="1021" alt="image" src="https://github.com/user-attachments/assets/94ba8191-9f80-4e01-8360-1995859da870" />
  
 <img width="1594" height="688" alt="image" src="https://github.com/user-attachments/assets/f7819ee4-e1f7-4a5c-96ca-e15464b472bc" />
 
 <img width="1738" height="1063" alt="image" src="https://github.com/user-attachments/assets/a041de0a-8670-4183-b1cc-574085196b47" />
 
- 这一张只用了AI代码中当某层没找到相应的Inode编码时情况的讨论，至于AI写的其它部分没太看懂也感觉太复杂了。  
## 最终境：算帝—— 递归与文件筛选  
### 【任务 4.1：敢问，路在何方？】  
**补全的代码**  
````  
#include <stdio.h>
#include <string.h>
#include <dirent.h>
#include <sys/stat.h>
#include <unistd.h>

// 递归查找指定文件，若找到则打印其相对路径
void find_file(const char *current_path, const char *target_name) {
    DIR *dir;
    struct dirent *entry;
    struct stat statbuf;

    if (!(dir = opendir(current_path))) {
        return; // 无法打开目录则跳过
    }

    while ((entry = readdir(dir)) != NULL) {
        // 忽略当前目录 . 和父目录 .. 防止无限死循环
        if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0) {
            continue;
        }

        char full_path[1024];
        snprintf(full_path, sizeof(full_path), "%s/%s", current_path, entry->d_name);

        if (stat(full_path, &statbuf) == -1) continue;

        // 请补全代码写在 TO DO 与 END OF TO DO 之间：
        // 要求：
        // 1. 检查当前 entry 的文件名是否与 target_name 匹配，若匹配则打印 full_path。
        // 2. 如果当前 entry 是一个目录，则需要对其进行深度优先的递归查找。
        // （提示：若(statbuf.st_mode & S_IFMT) == S_IFDIR则statbuf为目录文件）
        // TO DO
         if (strcmp(entry->d_name,target_name)==0)      //用strcmp函数来比较这两个字符串看是否完全相同。相同则打印出相对路径。  
        {
            printf("%s\n",full_path);              
        }

        if ((statbuf.st_mode & S_IFMT) == S_IFDIR)       //判断是否为目录，是则对其进行深度优先的递归查找
        {
            find_file(full_path,target_name);
        }


        // END OF TO DO
    }
    closedir(dir);
}
````     
---   
PS：这一题本来以为是要学习Linu系统中的各种命令，做完才发现是要研究命令背后的c语言基础（还得是c语言啊）。然后在这一题中主要也是采用了上网看视频查资料，加以AI辅助的模式来完成。还是学到了不少东西。比如发现了八进制的数的每一位其实都对应了这个数转换为二进制的连续三位。还有就是了解了像是`opendir` `readdir``closedir``strcmp`等函数的用法（上网搜到），还有的还有就是对Linux命令背后的原理有了更深入的了解和对某些特殊情况的讨论的了解。    

