
#c函数字符串的库源码的实现方式。
本文说明的是c函数字符串的库源码的实现方式。
其中函数声明方式：
__cdecl 是C 声明的缩写（Declaration），是C语言默认的函数调用方法：所有参数从右到左依次入栈，参数由调用者清除，即手动清栈。被调用函数不会要求调用者传递多少参数，调用者传递过多或者过少的参数，甚至完全不同的参数都不会产生编译阶段的错误。
_stdcall 是StandardCall的缩写，是C++的标准调用方式：所有参数从右到左依次入栈，如果是调用类成员的话，最后一个入栈的是this指针。这些堆栈中的参数由被调用的函数在返回清除，即自动清除，使用的指令是 retnX，X表示参数占用的字节数，CPU在ret之后自动弹出X个字节的堆栈空间。函数在编译的时候就必须确定参数个数，并且调用者必须严格的控制参数的生成，不能多或少，否则会编译出错。
##字符串比较
[cpp] view plain copy
1.int __cdecl strcmp (  
2.        const char * src,  
3.        const char * dst  
4.        )  
5.{  
6.        int ret = 0 ;  
7.        while( ! (ret = *(unsigned char *)src - *(unsigned char *)dst) && *dst)  
8.                ++src, ++dst;  
9.  
10.        if ( ret < 0 )  
11.                ret = -1 ;  
12.        else if ( ret > 0 )  
13.                ret = 1 ;  
14.  
15.        return( ret );  
16.}  

##字符串拷贝
[cpp] view plain copy
1.char * __cdecl strcpy(char * dst, const char * src)  
2.{  
3.        char * cp = dst;  
4.        while( *cp++ = *src++ ) ;               /* Copy src over dst */  
5.        return( dst );  
6.}  

##字符串拼接 
[cpp] view plain copy
1.char * __cdecl strcat (  
2.        char * dst,  
3.        const char * src  
4.        )  
5.{  
6.        char * cp = dst;  
7.        while( *cp )  
8.                cp++;                   /* find end of dst */  
9.        while( *cp++ = *src++ ) ;       /* Copy src to end of dst */  
10.        return( dst );                  /* return dst */  
11.}  



##字符串拷贝
[cpp] view plain copy
1.char * __cdecl strncpy (  
2.        char * dest,  
3.        const char * source,  
4.        size_t count  
5.        )  
6.{  
7.        char *start = dest;  
8.  
9.        while (count && (*dest++ = *source++))    /* copy string */  
10.                count--;  
11.  
12.        if (count)                              /* pad out with zeroes */  
13.                while (--count)  
14.                        *dest++ = '\0';  
15.  
16.        return(start);  
17.}  

##查找指定字符串
[cpp] view plain copy
1.char * __cdecl strstr (  
2.        const char * str1,  
3.        const char * str2  
4.        )  
5.{  
6.        char *cp = (char *) str1;  
7.        char *s1, *s2;  
8.        if ( !*str2 )  
9.            return((char *)str1);  
10.        while (*cp)  
11.        {  
12.                s1 = cp;  
13.                s2 = (char *) str2;  
14.  
15.                while ( *s1 && *s2 && !(*s1-*s2) )  
16.                        s1++, s2++;  
17.  
18.                if (!*s2)  
19.                        return(cp);  
20.  
21.                cp++;  
22.        }  
23.        return(NULL);  
24.}  

##查找字符串中最后出现的指定字符
[cpp] view plain copy
1.char * __cdecl strrchr (  
2.        const char * string,  
3.        int ch  
4.        )  
5.{  
6.        char *start = (char *)string;  
7.  
8.        while (*string++)                       /* find end of string */  
9.                ;  
10.                                                /* search towards front */  
11.        while (--string != start && *string != (char)ch)  
12.                ;  
13.        if (*string == (char)ch)                /* char found ? */  
14.                return( (char *)string );  
15.  
16.        return(NULL);  
17.}  