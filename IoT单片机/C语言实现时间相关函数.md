# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [C 库函数 - mktime()](https://www.runoob.com/cprogramming/c-function-mktime.html)
> [C语言实现将时间戳转换为年月日时分秒和将年月日时分秒转换为时间戳](https://www.cnblogs.com/it-book/p/9678135.html)
> [用C语言将微秒转换成年月日十分秒,求代码?](https://zhidao.baidu.com/question/556662421.html)
> [struct tm 和 time_t 时间和日期的使用方法](https://blog.csdn.net/u012260238/article/details/78961500)
> [C语言 __DATE__ __TIME__ Keil IAR 编译时间格式化](https://blog.csdn.net/WSCH_Prophet/article/details/79848992)

# 实现
最近在移植FatFs的`get_fattime`函数时，发现单片机没有相关时间函数，可以使用标准库的时间函数，但不知道能否编译通过，需要实现自己的`localtime`等函数，
```c
const char Days[12] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
void localtime(long time, struct tm_mb *t)
{
	unsigned int Pass4year;
	int hours_per_year;

	if(time < 0) {
		time = 0;
	}
	//取秒时间
	t->tm_sec=(int)(time % 60);
	time /= 60;
	//取分钟时间
	t->tm_min=(int)(time % 60);
	time /= 60;
	//取过去多少个四年，每四年有 1461*24 小时
	Pass4year=((unsigned int)time / (1461L * 24L));
	//计算年份
	t->tm_year=(Pass4year << 2) + 1970;
	//四年中剩下的小时数
	time %= 1461L * 24L;
	//校正闰年影响的年份，计算一年中剩下的小时数
	for (;;) {
		//一年的小时数
		hours_per_year = 365 * 24;
		//判断闰年
		if ((t->tm_year & 3) == 0) {
			//是闰年，一年则多24小时，即一天
			hours_per_year += 24;
		}
		if (time < hours_per_year) {
			break;
		}
		t->tm_year++;
		time -= hours_per_year;
	}
	//小时数
	t->tm_hour=(int)(time % 24);
	//一年中剩下的天数
	time /= 24;
	//假定为闰年
	time++;
	//校正闰年的误差，计算月份，日期
	if((t->tm_year & 3) == 0) {
			if (time > 60) {
				time--;
			} else {
				if (time == 60) {
				t->tm_mon = 1;
				t->tm_mday = 29;
				return ;
			}
		}
	}
	//计算月日
	for (t->tm_mon = 0; Days[t->tm_mon] < time; t->tm_mon++) {
		time -= Days[t->tm_mon];
	}
	t->tm_mday = (int)(time);
	return;
}

static int mon_yday[2][12] = {
	{0,31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334},
	{0,31, 60, 91, 121, 152, 182, 213, 244, 274, 305, 335},
};

int isleap(int year)
{
	return (year) % 4 == 0 && ((year) % 100 != 0 || (year) % 400 == 0);
}

long mktime(struct tm_mb dt)
{
	long result;
	int i =0;
	// 以平年时间计算的秒数
	result = (dt.tm_year - 1970) * 365 * 24 * 3600 + (mon_yday[isleap(dt.tm_year)][dt.tm_mon-1] + dt.tm_mday - 1) * 24 * 3600 + dt.tm_hour * 3600 + dt.tm_min * 60 + dt.tm_sec;
	// 加上闰年的秒数
	for(i = 1970; i < dt.tm_year; i++) {
		if(isleap(i)) {
			result += 24 * 3600;
		}
	}
	return result;
}
```

