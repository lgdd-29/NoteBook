# 函数

## 字符串处理函数

> 字符串处理函数都在<string.h>头文件中

### 获取字符串长度

```c
size_t strlen ( const char * str );
```

> strlen 函数用于返回指定字符串的长度。
>
> C 语言字符串的长度取决于结束符（'\0'）的位置。
>
> 一个字符串的长度指的是从起始位置到结束符的字符个数（不包含结束符本身）。
>
> 初学者很容易混淆字符串的长度和字符串数组的大小
>
> 函数返回的长度是size_t类型，其实就是unsigned类型

```c
char mystr[100] = "I love FishC.com!";
```

> 上边代码定义一个可以存放 100 个字符的数组，但 mystr 字符串只被初始化为包含 17 个字符的长度。因此，sizeof(mystr) 的结果是 100，而 strlen(mystr) 的结果则是 17。

### 拷贝字符串

```c
char *strcpy(char *dest, const char *src);
```

> dest	指向用于存放字符串的目标数组
>
> src	指向待拷贝的源字符串
>
> dest 长度必须要大于src,不然会出现问题
>
> 返回值是指向目标字符串的指针。

> strcpy 函数用于拷贝字符串，包含最后的结束符 '\0'。
>
> 为了避免溢出，必须确保用于存放的数组长度足以容纳待拷贝的字符串（注意：长度需要包含结束符 '\0'）。
>
> 源字符串和目标数组的位置不应该重叠。

```c
char *strncpy(char *dest, const char *src, size_t n);
```

> dest	指向存放字符串的目标数组
>
> src		指向待拷贝的源字符串
>
> n		指定拷贝的最大长度
>
> 返回值是指向目标字符串的指针。

> 和 [strcpy](http://bbs.fishc.com/thread-70518-1-1.html) 函数一样，strncpy(dest, src, n) 函数将拷贝源字符串的 n 个字符到目标数组中。如果源字符串的长度小于 n，那么就用 '\0' 填充额外的空间。如果源字符串的长度大于或等于 n，那么只有 n 个字符被拷贝到目标数组中（注意：这样的话将不会以结束符 '\0' 结尾）。
>
> 小甲鱼温馨提示：为了使该函数更“安全”，建议使用 dest[sizeof(dest) - 1] = '\0'; 语句确保目标字符串是以 '\0' 结尾。
>
> 源字符串和目标数组的位置不应该重叠。

### 连接字符串

```c
char *strncat(char *dest, const char *src, size_t n);
```

> dest 	指向用于存放字符串的目标数组，它应该包含一个字符串，并且提供足够容纳连接后的总字符串长度的空间（包含结束符 '\0'）
>
> src	指向待连接的源字符串，该参数不应该与 dest 参数指向的位置发生重叠
>
> n 指定待连接的源字符串的最大长度
>
> 返回值是指向目标字符串的指针。

> strncat 函数用于拷贝源字符串中的 n 个字符到目标数组的字符串后边，并在末尾添加结束符 '\0'。
>
> 如果源字符串的长度小于 n，那么不会像 strncpy 函数那样使用 '\0' 进行填充（但结束符 '\0' 还是有的）。
>
> 另外，目标数组中的原有的字符串并不算在 n 中。

### 比较字符串

```c
int strncmp(const char *s1, const char *s2, size_t n);
```

> strncmp 函数用于比较两个字符串的前 n 个字符。
>
> 该函数从第一个字符开始，依次比较每个字符的 ASCII 码大小，发现两个字符不相等或抵达结束符（'\0'）为止，或者前 n 个字符完全一样，也会停止比较。strncmp 函数用于比较两个字符串的前 n 个字符。
>
> 该函数从第一个字符开始，依次比较每个字符的 ASCII 码大小，发现两个字符不相等或抵达结束符（'\0'）为止，或者前 n 个字符完全一样，也会停止比较。

> s1		指向待比较的字符串 1
>
> s2		指向待比较的字符串 2
>
> n		指定待比较的字符数
>
> 返回一个整数表示两个字符串的关系：
>
> n<0		字符串 1 的字符小于字符串 2 对应位置的字符
>
> n=0		两个字符串的内容完全一致
>
> n>0		字符串 1 的字符大于字符串 2 对应位置的字符

## 内存管理函数

### malloc

```c
#include <stdlib.h>
void *malloc(size_t size);
```

malloc函数向系统申请分配size个字节的内存空间，并返回一个指向这块空间的指针。

> 返回的指针是空指针
>
> 如果函数调用成功，返回一个指向申请的内存空间的指针，由于返回类型是void 指针，所以它可以被住那换成任何类型的数据
>
> 如果函数调用失败，返回值是NULL。另外，如果size参数设置为0，返回值也可能是NULL，但这并不意味着函数调用失败。

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
	int* ptr;
	ptr = (int*)malloc(sizeof(int));  //将malloc返回的指针强制转换为int类型
	if (ptr == NULL)  //有可能分配内存会失败
	{
		printf("分配内存失败！\n");
		exit(1);
	}
	return 0;
}
```

### calloc

```c
void *calloc(size_t nmemb,size_t size);
```

> calloc函数在内存中动态地申请nmemb个长度为size的连续内存空间（即申请的总空间为nmemb * size)，这些内存空间全部被初始化为0
>
> calloc函数与malloc函数的一个重要区别是：
>
> calloc函数在申请完内存后，自动初始化该空间内存为0
>
> malloc函数不进行初始化操作，里面数据是随机的。

### free

```c
#include <stdlib.h>
void free(void *ptr);
```

> free函数释放ptr参数指向的内存空间。该内存空间必须是由malloc, calloc或realloc函数申请的。否则，该函数将导致未定义行为。如果ptr参数是NULL，则不执行任何操作。
>
> 注意：该函数并不会修改ptr参数的值，所以调用后它仍然指向原来的地方（变为非法空间）。

### memset

```c
menset()
```



### 内存块地址丢失

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
	int* ptr;
	int num = 123;
	ptr = (int*)malloc(sizeof(int));  //申请的内存只有ptr知道
	ptr = &num; //如果此时ptr指向其他地址，则之前申请的内存地址则丢失了
	free(ptr); //此时释放的是局部变量的地址，所以会出错。
	return 0;
}
```

## typedef

```c
#include <stdio.h>
typedef int integer;
int main(void)
{
	integer a;
	int b;
	a = 520;
	b = a;
	printf("%d", sizeof(a));
	return 0;
}
```

### 运用

一般与结构体一起用

```c
#include <stdio.h>
#include <stdlib.h>
typedef struct Date
{
	int year;
	int month;
	int day;
} DATE, *PDATE;
int main(void)
{
	PDATE date;
	date = (PDATE)malloc(sizeof(DATE));
	return 0;
}
```

> struct Date封装成了DATE
>
> struct Date* 封装成了DATE*
>
> 之后所有的struct Date都可以写成DATE
>
> 所有的struct Date* 都可以写成PDATE*

### 与#define的区别

#define是宏定义的直接替换，而typedef是对类型的封装。

例子如下：

> ```c
> #include <stdio.h>
> #define integer int
> int main(void)
> {
> 	unsigned integer a;
> 	a = -1;
> 	printf("a=%u\n", a);
> 	return 0;
> }
> ```
>
> 输出为a=4294967295
>
> ```c
> #include <stdio.h>
> typedef int integer;
> int main(void)
> {
> 	unsigned integer a;
> 	a = -1;
> 	printf("a=%u\n", a);
> 	return 0;
> }
> ```
>
> 会报错

> ```c
> #include <stdio.h>
> typedef int integer;
> typedef int* PTRINT;  //表示PTRINT是一个int*类型，指向int的指针
> int main(void)
> {
> 	integer a=520;
> 	PTRINT b, c;
> 	b = &a;
> 	c = b;
> 	printf("a=%p\n", c);
> 	return 0;
> }
> ```
>
> 此时b和c都是指向整形的指针
>
> ```c
> #include <stdio.h>
> typedef int integer;
> #define PTRINT int*
> int main(void)
> {
> 	integer a=520;
> 	PTRINT b, c;//=int *b,c;此时只有b是指针，c只是普通的整形
> 	b = &a;
> 	c = b;
> 	printf("a=%p\n", c);
> 	return 0;
> }
> ```
>
> typedef能改变类型的名称，而#define只是覆盖了名称，并不能从本质上改变类型的名称。

# 指针

## void指针

void指针称之为通用指针，可以指向任意类型的数据。

> 在不知道该指针要指向哪种数据的时候，可以用void指针。
>
> 在给确定的数据赋值后，最好将void指针强制转换为赋值后的指针类型

```c
int num=1024;
void *pv;
pv=pi;
printf("*pv=%d\n",*(int *)pv);//赋值的时候记得强制转换
```



## NULL指针

不指向任何数据的指针名为空指针（NULL指针）

> 如果一个指针不知知道初始化为什么地址时，先将它初始化为NULL。

## 指向指针的指针

存着指针的指针

```c
int num=520;
int *p=&num;
int **pp=&p;  
```

## 函数指针

```c
int *p(); //指针函数
int (*p)(); //函数指针，指向函数的指针
```

```c
#include <stdio.h>
int square(int);
int square(int num)   //函数
{
	return num * num;
}
int main()
{
	int num;
	int (*fp)(int);  //函数指针的定义，int是因为我们所指函数的返回值是个整形，后面的int是因为我们所指函数的参数是int
	printf("请输入一个整数：");
	scanf_s("%d", &num);
	fp = square;  //指向函数
	printf("%d *%d=%d\n", num, num, (*fp)(num));
	return 0;
}
```



不要返回局部变量的指针，因为局部变量的地址会变，此时指针会出现错误。

例如返回一个函数中的变量的指针，因为函数执行完，其中变量的内存会清空，此时指向变量地址的指针就会出现错误

## 函数指针作为参数

```c
#include <stdio.h>
int add(int, int);
int sub(int, int);
int calc(int(*fp)(int, int),int,int);  //calc总共有3个参数，一个是函数的指针，两个参数
int add(int num1, int num2)
{
	return num1 + num2;
}
int sub(int num1, int num2)
{
	return num1 - num2;
}
int calc(int(*fp)(int, int), int num1, int num2)
{
	return (*fp)(num1,num2);
}
int main()
{
	printf("3+5=%d\n", calc(add, 3, 5));
	printf("3-5=%d\n", calc(sub, 3, 5));
	return 0;
}
```

## 函数指针作为返回值

```c
//通过用户输入的加减来判断是否使用加函数还是减函数
#include <stdio.h>
int add(int, int);
int sub(int, int);
int calc(int(*fp)(int, int), int, int);  //根据select判断结果选择加函数还是减函数
int (*select(char ))(int, int);  //判断是加还是减
int add(int num1, int num2)
{
	return num1 + num2;
}
int sub(int num1, int num2)
{
	return num1 - num2;
}
int calc(int(*fp)(int, int), int num1, int num2)
{
	return (*fp)(num1, num2);
}
int (*select(char op))(int, int)  //本质上还是一个函数指针  int(*fp)(int,int) 将fp改为select(char op)
{
	switch (op)
	{
		case '+': return add;
		case '-': return sub;
	}
}
int main()
{
	int num1, num2;
	char op;
	int (*fp)(int, int);
	printf("请输入一个式子：");
	scanf_s("%d %c %d", &num1, &op, &num2);
	fp = select(op);
	printf("%d %c %d = %d", num1, op, num2, calc(fp, num1, num2));
	return 0;
}
```

# 递归

递归就是在函数中不断调用自身

递归必须要有结束条件，否则程序将崩溃

```c
//计算一个正整数的阶乘
#include <stdio.h>
long fact(int);
long fact(int num)  //递归函数
{
	long result;
	if (num > 0)
	{
		result = num * fact(num - 1);  //不断调用自身，直到跳过if不执行自身，才会有返回值，然后逐层返回
	}
	else
	{
		result = 1;
	}
	return result;
}
int main()
{
	int num;
	printf("请输入一个正整数：");
	scanf_s("%d", &num);
	printf("%d的阶乘为%d", num, fact(num));
	return 0;
}
```

## 汉诺塔

```c
#include <stdio.h>
void hanoi(int, char, char, char);
void hanoi(int n, char x, char y, char z)  //汉诺塔的递归函数
{
	if (n == 1)
	{
		printf("%c --->%c\n", x, z);
	}
	else
	{
		hanoi(n - 1, x, z, y);
		printf("%c --->%c\n", x, z);
		hanoi(n - 1, y, x, z);
	}
}
int main(void)
{
	int n;
	printf("请输入汉诺塔的层数：");
	scanf_s("%d", &n);
	hanoi(n, 'X', 'Y', 'Z');

	return 0;
}
```

## 快速排序

操作步骤是先找一个基准点，然后比基准点小的排在左边，比基准点大的排在右边，然后左边和右边再各选一个基准点，再如此操作，直到没有元素。

```c
#include <stdio.h>
void quick_sort(int array[], int, int);
void quick_sort(int array[], int left, int right)  //快速排序的函数
{
	int i = left, j = right; //left是左边的开始，right是右边的开始
	int temp;
	int pivot;
	pivot = array[(left + right) / 2]; //基准点的位置，此时是数组的中间位置
	while(i <= j)
	{
		while (array[i] < pivot)  //从左到右找到大于等于基准点的元素
		{
			i++;
		}
		while (array[j] > pivot)//从右到左找到大于等于基准点的元素
		{
			j--;
		}
		if (i <= j)
		{
			temp = array[i];
			array[i] = array[j];
			array[j] = temp;
			i++;
			j--;
		}
	}
	if (left < j)
	{
		quick_sort(array, left, j);  //此时j不是原来函数right的值，而是新函数的right并且j<right,右边界的范围减小了。
	}
	if (i < right)
	{
		quick_sort(array, i, right); //此时i不是原来函数left的值，范围减小，调用自身
	}
}
int main(void)
{
	int array[] = { 73,108,111,118,101,70,105,115,104,67,46,99,111,109 };
	int i, length;
	length = sizeof(array) / sizeof(array[0]);
	quick_sort(array,0,length-1);
	printf("排序后的结果是：");
	for (i = 0; i < length; i++)
	{
		printf("%d	", array[i]);
	}
	putchar('\n');
	return 0;
}
```



# 链表

## 单链表

![image-20240715161209075](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/image-20240715161209075.png)

单链表有两部分，一部分是信息域，用于存放信息。另一部分是指针域，用来指向下一个单链表，知道最后一个节点指向NULL。

> 不需要紧密排放，通过指针连接

```c
#include <stdio.h>
#include <stdlib.h>
struct Student  //定义结构体
{
	char name[30];
	int year;
	struct Student* next;  //定义结构体指针
}student;
int main(void)
{
	struct Student * p, * head, * tail;  //定义节点，为什么不能用student *p是因为此时编译器还不知道student是什么，因为结构体不完善
	p = (struct Student*)malloc(sizeof(struct Student));  //为了使p能等于结构体，所以分配的内存要和结构体一致
	head = p;   //刚开始头节点也在p
	tail = p;   //尾节点也在p
	head->next = NULL;  //此时只有一个节点，所以头节点和尾节点的下一个都是NULL
	for (int i = 0; i < 3; i++)
	{
		p = (struct Student*)malloc(sizeof(struct Student)); //因为节点的位置在变化，所以到达一个新的节点就要内存分配
		printf("请输入姓名：");
		scanf("%s", &p->name);
		printf("请输入年龄：");
		scanf("%d", &p->year);
		tail->next = p; //尾节点指针指向p
		tail = p;		//尾节点等于p
		tail->next = NULL;//尾节点的指针指向NULL
	}
	p = head->next; //p是要回到第一个装满内容的节点上
	while (p != NULL)
	{
		printf("名字：%s\n", p->name);
		printf("年龄：%d\n", p->year);
		p = p->next;
	}
	return 0;
}
```

> 头节点一般没有东西，只是为了表示这是头位置
>
> 内存块的地址都在，可以通过指针访问，例如四个节点的链表中：头节点的head->next->next->next的位置就是尾节点，所以说我们只知道头节点和尾节点的地址位置，中间节点的地址我们是不知道的，只能通过头节点结构体中的结构体指针才知道中间节点的地址。
>
> p节点装满东西后，然后tial的下一个指针指向p，然后使tial等于p。然后这样就实现了p的接入，然后再给p申请一块新内存，再接入tail形成链表。

```c
//用单链表的方式录入书籍信息并打印
#include <stdio.h>
#include <stdlib.h>
struct Book
{
	char title[128];
	char author[40];
	struct Book* next;
};
void getInput(struct Book* book)
{
	printf("请输入书名:");
	scanf("%s", book->title);
	printf("请输入作者：");
	scanf("%s", book->author);
}
void addBook(struct Book ** library)
{
	struct Book* book, *temp;
	book = (struct Book*)malloc(sizeof(struct Book));
	if (book == NULL)
	{
		printf("内存分配失败\n");
		exit(1);
	}getInput(book);
	if (*library != NULL)
	{
		temp = *library;
		*library = book;
		book->next = temp;
	}
	else
	{
		*library = book;
		book->next = NULL;
	}
}
void printLibrary(struct Book* library)
{
	struct Book* book;
	int count = 1;
	book = library;
	while (book!=NULL)
	{
		printf("Book%d:", count);
		printf("书名：%s", book->title);
		printf("作者：%s", book->author);
		book = book->next;
		count++;
	}
}
void releaseLibrary(struct Book* library)
{
	while (library != NULL)
	{
		library = library->next;
	}
}

int main()
{
	struct Book* library = NULL;
	int ch;
	while (1)
	{
		printf("请问是否需要录入书籍信息(Y/N):");
		do
		{
			ch = getchar();
		} while (ch != 'Y' && ch != 'N');
		if (ch == 'Y')
		{
			addBook(&library);
		}
		else
		{
			break;
		}
	}
	printf("请问是否需要打印图书信息(Y/N):");
	do
	{
		ch = getchar();
	}while(ch != 'Y' && ch != 'N');
	if (ch == 'Y')
	{
		printLibrary(library);
	}
	releaseLibrary(library);
	addBook(&library);
	return 0;
}
```



```c
#include <stdio.h>
#include <stdlib.h>
struct Student
{
	char name[40];
	int year;
	char answear1;
	char answear2;
	char date1[20];
	char date2[20];
	struct Student *next;
};
int main(void)
{
	char Answear;
	struct Student *p,*head,*tail;
	p = (struct Student*)malloc(sizeof(struct Student));
	head = p;
	tail = p;
	printf("是否需要录入信息(Y/N)：");
	scanf("%c", &Answear); getchar();
	while (Answear == 'Y')
	{
		p = (struct Student*)malloc(sizeof(struct Student));
		printf("请问姓名是：");
		scanf("%s", &p->name); getchar();
		printf("请问年龄是：");
		scanf("%d", &p->year);getchar();
		printf("请问是否接种过疫苗(Y/N):");
		scanf("%c", &Answear); getchar();
		p->answear1 = Answear;
		if (Answear == 'Y')
		{
			printf("请输入第一针疫苗的接种的日期(yyyy-mm-dd):");
			scanf("%s", &p->date1); getchar();
			printf("请问是否接种第二针疫苗(Y/N):");
			scanf("%c", &Answear); getchar();
			p->answear2 = Answear;
			if (Answear == 'Y')
			{
				printf("请输入第二针疫苗的接种的日期(yyyy-mm-dd):");
				scanf("%s", &p->date2); getchar();
			}
			else
			{
				printf("请尽快接种第二针疫苗!\n");
			}
		}
		else
		{
			printf("请尽快接种疫苗！\n");
		}
		printf("\n是否需要录入信息(Y/N)：");
		scanf("%c", &Answear); getchar();
		tail->next = p;
		tail = p;
		tail->next = NULL;
	}
	p = head->next;
	if(Answear=='N')
	{
		printf("请问是否需要打印已录入数据(Y/N):");
		scanf("%c", &Answear); getchar();
		if (Answear == 'Y')
		{
			while (p != NULL)
			{
				printf("\n姓名是：%s\n", p->name);
				printf("年龄是：%d\n", p->year);
				if (p->answear1=='Y')
				{
					printf("第一针疫苗接种日期：%s\n", p->date1);
					if (p->answear2=='Y')
					{
						printf("第二针疫苗接种日期：%s\n", p->date2);
					}
					else printf("未接种第二针疫苗！\n");
				}
				else printf("未接种疫苗\n");
				p = p->next;
			}
		}
	}
	return 0;
}
```

## 搜索节点

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
struct Book
{
	char name[40];
	int price;
	struct Book* next;
};
struct Book* searchBook(struct Book* head, char* target)  //搜索函数，参数是结构体指针和用户输入的字符串指针
{
	struct Book *book=head->next;   //新建一块内存放在head下一个节点上
	while (book != NULL)
	{
		if (!strcmp(book->name, target))  //比较函数，如果一致则返回0，所以要加！倒置
		{
			return book;//返回book的结构体指针
		}
		book = book->next;  //下一次循环，直到到尾指针
	}
	return NULL;
}
int main(void)
{
	struct Book* p, * head, * tail;
	char *target;
	target = (char*)malloc(41*sizeof(char)); //分配了41个char类型大小的内存
	p = (struct Book*)malloc(sizeof(struct Book));
	head = p;
	tail = p;
	tail->next = NULL;
	for (int i = 0; i < 1; i++)
	{
		p = (struct Book*)malloc(sizeof(struct Book));
		printf("输入名字：");
		scanf("%39s", p->name);
		printf("输入价格：");
		scanf("%d", &p->price);
		tail->next = p;  //p并没有插进去，而是通过tail下一个野指针对p的复制形成的链表
		tail = p;
		tail->next = NULL;
	}
	printf("搜索：");
	scanf("%39s",target);
	p = searchBook(head, target);
	if (p != NULL) {
		printf("名字为：%s的价格为%d", p->name, p->price);
	}
	else
	{
		printf("没有找到名为%s的书\n", target);
	}
	while (head != NULL)
	{
		p = head;
		head = head->next;
		free(p);
	}
	free(target);
	return 0;
}
```

## 内存池

## 菜单

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
struct System
{
	char name[40];
	char number[50];
	struct System* next;
};
void Display(void);
struct System* Addperson(struct System**);
void Displaytast(struct System*);
struct System* findPerson(struct System*);
void changePerson(struct System*,struct System**);
void Free(struct System*);
struct System* delPerson(struct System** ,struct System**,struct System *);
void displayContacts(struct System*);
int main(void)
{
	int Answear, i = 0;
	struct System* head, * p, * tail;
	head = tail  = (struct System*)malloc(sizeof(struct System));
	p = NULL;
	tail->next = NULL;
	while (1)
	{
		Display();
		printf("请输入命令指令：");
		scanf("%d", &Answear);
		switch (Answear)
		{
		case 1:
			p=Addperson(&tail);
			break;
		case 2:
			p= findPerson(head);
			if(p!=NULL) printf("电话号码为：%s\n", p->number);
			break;
		case 3:
			p = findPerson(head);
			if(p!=NULL) changePerson(head,&p);
			break;
		case 4:
			p = findPerson(head);
			if (p != NULL) p=delPerson(&head, &tail,p);
			break;
		case 5:
			displayContacts(p);
			break;
		case 6:
			Displaytast(head);
			break;
		case 7:
			i = 1;
			break;
		default:
			printf("无相关指令，请重新输入：");
			break;
		}
		if (i == 1) break;
	}
	Free(head);
	return 0;
}
void displayContacts(struct System* p)
{
	if (p == NULL) printf("当前无联系人!\n");
	else
	{
		printf("当前联系人为%s\n电话号码为%s\n", p->name, p->number);
	}
}
struct System* delPerson(struct System** head, struct System** tail,struct System* p)
{
	struct System** now=head;
	while ((*now)->next != p)
	{
		now = &((*now)->next);
	}
	if (p->next == NULL)
	{
		*tail = *now;
	}
	(*now)->next = p -> next;
	free(p);
	return NULL;
}
void Free(struct System* head)  //程序最后释放内存
{
	struct System* temp;
	while (head != NULL)
	{
		temp = head;
		head = head->next;
		free(temp);
	}
}
void changePerson(struct System* head,struct System **p)
{
	printf("请输入新的号码：");
	scanf("%s", (*p)->number);
}
struct System* findPerson(struct System* head)
{
	struct System* person;
	char* target;
	target = (char*)malloc(40 * sizeof(char));
	printf("请输入名字搜索：");
	scanf("%39s", target);
	person = head->next;
	while (person != NULL)
	{
		if (!strcmp(person->name, target))
		{
			return person;
		}
		person = person->next;
	}
	printf("没有找到该联系人!\n");
	return NULL;
}
void Displaytast(struct System* head)
{
	struct System* i;
	i = (struct System*)malloc(sizeof(struct System));
	i = head->next;
	while (i != NULL)
	{
		printf("\n名字：%s\n", i->name);
		printf("电话：%s\n", i->number);
		i = i->next;
	}
	if (head->next == NULL) printf("联系人为空！\n");
	free(i);
}
struct System * Addperson(struct System** tail)
{
	struct System* person;
	person = (struct System*)malloc(sizeof(struct System));
	printf("请输入名字：");
	scanf("%s", &person->name);
	printf("请输入电话：");
	scanf("%s", &person->number);
	(*tail)->next = person;
	(*tail) = person;
	(*tail)->next = NULL;
	return (*tail);
}
void Display(void)
{
	printf("\n\n");
	printf("| 欢迎使用通信录管理程序  |\n");
	printf("|                         |\n");
	printf("|--- 1：插入新的联系人 ---|\n");
	printf("|                         |\n");
	printf("|--- 2：查找已有联系人 ---|\n");
	printf("|                         |\n");
	printf("|--- 3：更改已有联系人 ---|\n");
	printf("|                         |\n");
	printf("|--- 4：删除已有联系人 ---|\n");
	printf("|                         |\n");
	printf("|--- 5：显示当前联系人 ---|\n");
	printf("|                         |\n");
	printf("|--- 6：显示所有联系人 ---|\n");
	printf("|                         |\n");
	printf("|--- 7：退出通信录程序 ---|\n");
	printf("\n\n");
}
```

> ```c
> void Addperson(struct System** tail)
> ```
>
> 这句话定义了一个名为`Addperson`的函数，其目的是在链表的末尾添加一个新的`System`结构体节点，并更新指向链表尾部的指针。以下是各个部分的详细解释：
>
> - `void`：这表示`Addperson`函数不返回任何值。
> - `Addperson`：这是函数的名称。
> - `(struct System** tail_ptr)`：这是函数的参数列表，它包含一个参数`tail_ptr`。
>   - `struct System*`：这指定了参数`tail_ptr`的类型，它是一个指向`System`结构体的指针。
>   - `* tail_ptr`：这里的星号`*`表示`tail_ptr`本身是一个指针，但它不是指向`System`结构体的普通指针，而是一个指向`System`结构体指针的指针，也就是双重指针。
>   整体来看，`struct System** tail_ptr`意味着`Addperson`函数接收一个指向`System`结构体指针的指针作为参数。这个参数允许函数修改调用者提供的`System`结构体指针，从而更新链表的尾部。这是必要的，因为如果只是按值传递一个普通的指针，那么函数内部对指针的任何修改都不会影响到函数外部的原始指针。通过使用双重指针，`Addperson`函数能够直接修改原始的尾指针，使其指向新添加的节点。
>
> ```c
> void Addperson(struct System** tail)
> {
> 	struct System* person;
> 	person = (struct System*)malloc(sizeof(struct System));
> 	printf("请输入名字：");
> 	scanf("%s", &person->name);
> 	printf("请输入电话：");
> 	scanf("%s", &person->number);
> 	(*tail)->next = person;
> 	(*tail) = person;
> 	(*tail)->next = NULL;
> }
> 大致意思是：如果你只是要使用结构体指针，则system *tail就可以了
> 如果你在函数中想要修改tail的值，则需要使用system **tail中指向结构体指针的指针lai
> ```
>

> ```c
> void delPerson(struct System** head, struct System** tail,struct System* p)
> {
> 	struct System** now=head;
> 	while ((*now)->next != p)
> 	{
> 		now = &((*now)->next);
> 	}
> 	if (p->next == NULL)
> 	{
> 		*tail = *now;
> 	}
> 	(*now)->next = p -> next;
> 	free(p);
> }
> ```
>
> 如果删除的是尾节点，那么要更新尾节点的位置
>
> ```c
> now = &((*now)->next);
> ```
>
> 在给出的代码行 `now = &((*now)->next);` 中，`now` 是一个指向指针的指针，即 `struct System**` 类型。这样做的原因是为了在单向链表中正确地遍历和修改节点指针。
> 以下是为什么需要这样做的详细解释：
> 1. **遍历链表**：
>    - 在单向链表中，每个节点包含数据和指向下一个节点的指针。为了遍历链表，我们需要一个指针来跟踪当前节点。在这个函数中，`now` 是一个指向当前节点指针的指针（`struct System**`），这意味着 `*now` 是当前节点的指针（`struct System*`）。
> 2. **修改前一个节点的指针**：
>    - 当我们想要删除链表中的一个节点时，我们需要修改该节点前一个节点的 `next` 指针，使其指向要删除节点的下一个节点。如果我们仅仅使用一个指向当前节点的指针（`struct System*`），我们无法直接修改前一个节点的 `next` 指针，因为我们没有前一个节点的引用。
> 3. **通过指针的地址来修改指针**：
>    - 使用 `now = &((*now)->next);` 的目的是让 `now` 指向当前节点的 `next` 指针的地址。这样，当我们发现要删除的节点时，我们可以通过 `*now` 来修改前一个节点的 `next` 指针，因为我们现在持有它的地址。
>    具体步骤如下：
> - `*now` 解引用 `now`，得到当前节点的指针。
> - `(*now)->next` 获取当前节点的下一个节点的指针。
> - `&((*now)->next)` 获取当前节点的 `next` 指针的地址。
> - `now = &((*now)->next);` 将 `now` 更新为当前节点的 `next` 指针的地址。
> 通过这种方式，当找到要删除的节点时，`now` 指向的是要删除节点的前一个节点的 `next` 指针的地址，这样我们就可以通过 `*now` 来修改它，完成节点的删除操作。
>
> ```c
> *tail = *now;
> ```
>
> 在您提供的代码片段中，`*tail = *now;` 这行代码的目的是更新链表的 `tail` 指针。这里是详细解释：
> - `now` 是一个指向指针的指针（`struct System**`），它当前指向链表中要删除节点的前一个节点的 `next` 指针的地址。
> - `*now` 是对 `now` 的解引用，因此它表示要删除节点的前一个节点的 `next` 指针，即 `struct System*` 类型。
> 当以下条件成立时：
> ```c
> if (p->next == NULL)
> ```
> 这表示要删除的节点 `p` 是链表的最后一个节点（因为它没有下一个节点）。在这种情况下，要删除的节点 `p` 刚好是 `tail` 指向的节点。
> 为了保持 `tail` 指针的正确性，您需要将 `tail` 更新为指向新的最后一个节点。由于要删除的是最后一个节点，新的最后一个节点应该是要删除节点的前一个节点。因此，代码执行以下操作：
> ```c
> *tail = *now;
> ```
> - `*tail` 是对 `tail` 的解引用，它允许您修改 `tail` 指针指向的值。
> - `*now` 是要删除节点的前一个节点。
> 因此，`*tail = *now;` 实际上是在说：“将 `tail` 更新为指向要删除节点的前一个节点。” 这确保了在删除最后一个节点之后，`tail` 仍然正确地指向链表的最后一个节点。
>
> tail是指向结构体指针的指针，如果要更新tail结构体的指向，只需要一层指针，所以要减掉一层指针，*tail= *now才可以。

# 结构体

## 形式

struct 结构体名称	结构体变量名

```c
struct Book //结构体名称
{
    char title[128];
    char author [40];
    float price;
} book;  //结构体变量名
```

> 要访问结构体成员，我们需要引入一个新的运算符--点号（.）运算符。比如book.title就是引用book结构体的title成员，它是一个字符数组；而book.price则是引用book结构体的price成员，它是一个浮点型的变量。

## 初始化

```c
struct Book book={    //还要带上结构体变量名名称
    .price = 48.8,  //是逗号
    .data = 20171111
}
```

## 结构体数组

```c
struct Nume //结构体名称
{
    //结构体成员
} 数组名[长度]；
```

## 结构体指针

```c
struct Book * pt;
pt = &book;  //要用&才能取得book结构体的地址
```

```c
通过结构体指针访问结构体成员又两种方法：
(*结构体指针).成员名
结构体指针->成员名
```

```c
#include <stdio.h>
#include <stdio.h>
struct Data  //结构体定义
{
	int Month;
	int Day;
} data;
int main(void)
{
	struct Data* pt;  //结构体指针定义
	pt = &data;   //结构体指针指向data结构体，此时指向的是结构体的结构体变量名而不是结构体名称
	(*pt).Month = 7;
	pt->Day = 16;
	return 0;
}
```

## 结构体变量传递

```c
#include <stdio.h>
int main(void)
{
	struct Test
	{
		int x;
		int y;
	}t1,t2;
	t1.x = 3;
	t1.y = 4;
	t2 = t1;  //可以直接将结构体变量等于另一个结构体变量
	printf("t2.x=%d,t2.y=%d", t2.x, t2.y);
	return 0;
}
```

```c
#include<stdio.h>
struct Data   //日期的
{
	int year;
	int month;
	int day;
};
struct Book
{
	char title[128];
	char author[40];
	float price;
	struct Data data;
	char publisher[40];
}b1,b2;
struct Book getInput(struct Book book)
{
	printf("请输入书名：");
	scanf("%s", book.title);
	printf("请输入作者：");
	scanf("%s", book.author);
	printf("请输入售价：");
	scanf("%f", &book.price);
	printf("请输入出版日期：");
	scanf("%d-%d-%d", &book.data.year, &book.data.month, &book.data.day);
	printf("请输入出版社：");
	scanf("%s", book.publisher);
	return book;
};
void printBook(struct Book book)
{
	printf("书名：%s\n", book.title);
	printf("作者：%s\n", book.author);
	printf("售价：%.2f\n", book.price);
	printf("出版日期：%d-%d-%d\n", book.data.year, book.data.month, book.data.day);
	printf("出版社：%s\n", book.publisher);
}
int main(void)
{
	printf("请录入第一本书的信息：\n");
	b1=getInput(b1);
	putchar('\n');
	printf("请录入第二本书的信息：\n");
	b2 = getInput(b2);
	printf("\n\n录入完毕，现在开始打印验证...\n\n");
	printf("打印第一本书的信息：");
	printBook(b1);
	putchar('\n');
	printf("打印第二本书的信息：");
	printBook(b2);
	return 0;
}
```

> 结构体变量的传递执行的效率很低，所以一般用指针指向结构体

## 结构体指针

```c
#include<stdio.h>
struct Data
{
	int year;
	int month;
	int day;
};
struct Book
{
	char title[128];
	char author[40];
	float price;
	struct Data data;
	char publisher[40];
}b1,b2;
void getInput(struct Book *book)  //改为无返回值，并且传进来的是book结构体变量的地址，这样就能用book->成员名来使用结构体了
{								//因为只有知道了book的地址，才能使用book->成员名这种指针的方式
	printf("请输入书名：");
	scanf("%s", book->title);
	printf("请输入作者：");
	scanf("%s", book->author);
	printf("请输入售价：");
	scanf("%f", &book->price);
	printf("请输入出版日期：");
	scanf("%d-%d-%d", &book->data.year, &book->data.month, &book->data.day);
	printf("请输入出版社：");
	scanf("%s", book->publisher);
};
void printBook(struct Book *book)  //这里也是同理，传进去的是结构体指针
{
	printf("书名：%s\n", book->title);
	printf("作者：%s\n", book->author);
	printf("售价：%.2f\n", book->price);
	printf("出版日期：%d-%d-%d\n", book->data.year, book->data.month, book->data.day);
	printf("出版社：%s\n", book->publisher);
}
int main(void)
{
	printf("请录入第一本书的信息：\n");
	getInput(&b1);
	putchar('\n');
	printf("请录入第二本书的信息：\n");
	getInput(&b2);
	printf("\n\n录入完毕，现在开始打印验证...\n\n");
	printf("打印第一本书的信息：");
	printBook(&b1);
	putchar('\n');
	printf("打印第二本书的信息：");
	printBook(&b2);
	return 0;
}
```

## 结构体指针函数

````c
在C语言中，`struct Book* searchBook(struct Book* book, char* target)` 这行代码声明了一个函数`searchBook`，这个函数的目的是在给定的书籍列表中搜索特定的书籍。下面是对这个函数声明的详细解释：
- `struct Book*`：这表示函数返回一个指向`Book`结构体的指针。`Book`是一个结构体类型，可能包含了关于书籍的各种信息，比如书名、作者、ISBN等。
- `searchBook`：这是函数的名称，表明这个函数执行的是搜索书籍的操作。
- `book`：这是函数的第一个参数，它是一个指向`Book`结构体的指针，可能指向一个书籍数组的第一个元素，这个数组包含了所有要搜索的书籍。
- `char* target`：这是函数的第二个参数，它是一个字符指针，指向一个字符串，这个字符串是搜索的目标，比如书名或者作者名。
函数的工作流程可能是这样的：
1. 函数遍历由`book`参数指向的书籍列表。
2. 对于列表中的每一本书，函数会比较书籍的某个属性（比如书名）与`target`指向的字符串是否匹配。
3. 如果找到了匹配的书籍，函数将返回一个指向该书籍结构体的指针。
4. 如果没有找到匹配的书籍，函数可能返回`NULL`，表示搜索失败。
以下是一个简化的例子，展示了`searchBook`函数可能的实现：
```c
#include <stdio.h>
#include <string.h>
// 假设Book结构体定义如下
struct Book {
    char title[100];
    char author[100];
    int id;
};
// searchBook函数的实现
struct Book* searchBook(struct Book* book, int numBooks, char* target) {
    for (int i = 0; i < numBooks; i++) {
        if (strcmp(book[i].title, target) == 0) {
            return &book[i]; // 找到匹配的书，返回其指针
        }
    }
    return NULL; // 没有找到匹配的书
}
// 使用searchBook函数的例子
int main() {
    struct Book library[] = {
        {"The Great Gatsby", "F. Scott Fitzgerald", 1},
        {"1984", "George Orwell", 2},
        {"To Kill a Mockingbird", "Harper Lee", 3}
    };
    int numBooks = sizeof(library) / sizeof(library[0]);
    
    char* target = "1984";
    struct Book* foundBook = searchBook(library, numBooks, target);
    
    if (foundBook != NULL) {
        printf("Book found: %s\n", foundBook->title);
    } else {
        printf("Book not found.\n");
    }
    
    return 0;
}
```
在这个例子中，`searchBook`函数通过书名来搜索书籍，并返回找到的书籍的指针。如果没有找到，则返回`NULL`。注意，为了使这个函数正常工作，你需要传入书籍数组的大小`numBooks`。

````



## 结构体分配内存

```c
#include<stdio.h>
#include<stdlib.h>
struct Data
{
	int year;
	int month;
	int day;
};
struct Book
{
	char title[128];
	char author[40];
	float price;
	struct Data data;
	char publisher[40];
};
void getInput(struct Book *book)
{
	printf("请输入书名：");
	scanf("%s", book->title);
	printf("请输入作者：");
	scanf("%s", book->author);
	printf("请输入售价：");
	scanf("%f", &book->price);
	printf("请输入出版日期：");
	scanf("%d-%d-%d", &book->data.year, &book->data.month, &book->data.day);
	printf("请输入出版社：");
	scanf("%s", book->publisher);
};
void printBook(struct Book *book)
{
	printf("书名：%s\n", book->title);
	printf("作者：%s\n", book->author);
	printf("售价：%.2f\n", book->price);
	printf("出版日期：%d-%d-%d\n", book->data.year, book->data.month, book->data.day);
	printf("出版社：%s\n", book->publisher);
}
int main(void)
{
	struct Book* b1, * b2;  //定义了b1,b2的指针
	b1 = (struct Book * )malloc(sizeof(struct Book)); //给b1,b2申请内存
	b2 = (struct Book*)malloc(sizeof(struct Book));
	if (b1 == NULL || b2 == NULL)
	{
		printf("内存分配失败！");
		exit(1);
	}
	printf("请录入第一本书的信息：\n");
	getInput(b1);
	putchar('\n');
	printf("请录入第二本书的信息：\n");
	getInput(b2);
	printf("\n\n录入完毕，现在开始打印验证...\n\n");
	printf("打印第一本书的信息：");
	printBook(b1);
	putchar('\n');
	printf("打印第二本书的信息：");
	printBook(b2);

	free(b1);
	free(b2);
	return 0;
}
```

# 枚举

枚举（Enum）在C语言中是一种用户定义的数据类型，它由一组命名的整数常量组成。枚举在C语言中非常有用，主要体现在以下几个方面：
1. **代码可读性增强**：枚举通过名称来定义一组整型常量，这使得代码更加易于理解和维护。例如，用枚举来表示星期可以让人一目了然，而不是使用无意义的数字。
2. **类型安全**：使用枚举可以限制变量的取值范围，从而避免赋予变量不合法的值。例如，一个表示操作状态的枚举只能被赋予定义中的某个值，这有助于减少程序错误。
3. **简化比较**：由于枚举实际上是一组整数，因此可以很方便地用来进行比较操作，如判断某个枚举变量是否等于某个特定的枚举值。
4. **方便遍历**：枚举的值默认从0开始，并以此递增，这使得遍历枚举值变得容易。
5. **清晰的默认值**：如果没有为枚举变量显式赋值，它们会自动从0开始赋值，这为默认行为提供了方便。
6. **定义状态机**：枚举经常用来定义状态机中的状态，使得状态的转换清晰且易于管理。
7. **组织相关常量**：当一组相关的常量需要一起使用时，枚举可以作为一个组织工具，将它们集合在一起。
以下是一个简单的枚举示例：
```c
#include <stdio.h>
// 定义一个表示星期的枚举
enum Weekday {
    Sunday,    // 默认为 0
    Monday,    // 默认为 1
    Tuesday,   // 默认为 2
    Wednesday,
    Thursday,
    Friday,
    Saturday
};
int main() {
    enum Weekday today = Wednesday;
    printf("Day %d\n", today); // 输出 Day 3
    // 使用枚举进行条件判断
    if (today == Wednesday) {
        printf("Today is Wednesday\n");
    }
    return 0;
}
```
在这个例子中，枚举`Weekday`定义了一个星期的七天，使得代码在表示和检查日期时更加清晰和直观。



## 枚举定义状态机

```c
#include <stdio.h>
// 定义状态机的可能状态
typedef enum {
    STATE_START,
    STATE_STOP,
    STATE_RUNNING,
    STATE_PAUSED
} State;
// 定义触发状态转移的事件
typedef enum {
    EVENT_START,
    EVENT_STOP,
    EVENT_PAUSE,
    EVENT_RESUME
} Event;
// 状态转移函数
State stateTransition(State current, Event event) {
    switch (current) {
    case STATE_START:
        if (event == EVENT_START) {
            return STATE_RUNNING;
        }
        break;
    case STATE_RUNNING:
        if (event == EVENT_STOP) {
            return STATE_STOP;
        }
        else if (event == EVENT_PAUSE) {
            return STATE_PAUSED;
        }
        break;
    case STATE_PAUSED:
        if (event == EVENT_RESUME) {
            return STATE_RUNNING;
        }
        else if (event == EVENT_STOP) {
            return STATE_STOP;
        }
        break;
    case STATE_STOP:
        if (event == EVENT_START) {
            return STATE_RUNNING;
        }
        break;
    }
    // 如果没有合适的转移，返回当前状态
    return current;
}
int main() {
    // 初始状态为STATE_START
    State currentState = STATE_START;
    // 模拟状态转移
    currentState = stateTransition(currentState, EVENT_START);
    printf("Current State: RUNNING\n");
    currentState = stateTransition(currentState, EVENT_PAUSE);
    printf("Current State: PAUSED\n");
    currentState = stateTransition(currentState, EVENT_RESUME);
    printf("Current State: RUNNING\n");
    currentState = stateTransition(currentState, EVENT_STOP);
    printf("Current State: STOP\n");
    return 0;
}
```

# 位操作

## 逻辑位运算符

![](C:\学习\笔记\照片\微信图片编辑_20240720150049.jpg)

### ~

将二进制中的10取反

### &

两个二进制的各个值进行对比，只有同时为1才为1

​    a=1111	1010

​    b=1010	1111

-----------------

a&b=1010	1010

> 只要有0就为0

### |

两个二进制的各个值进行对比，只有同时为0才为0

​    a=1111	1010

​    b=1010	1111

-----------------

a|b =1111	1111

> 只要有1就为1

### ^

不同为1，相同为0

​    a=1111	1010

​    b=1010	1111

-----------------

a^b =0101	0101

## 移位运算符

### 左移位运算符

11001010<<2=00101000

### 右移位运算符

11001010>>2=00110010
