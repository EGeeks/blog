# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 运算符`**`
表示次幂。

# 系统函数`$clog2`
> [AR# 44586 13.2 Verilog $clog2 function implemented improperly](https://china.xilinx.com/support/answers/44586.html)

求一个数以2为底的对数，并向上取整，用于计算需要多少位宽。
```verilog
module tb;
parameter A = clog2(325);

function integer clog2;
input integer value;
begin
value = value-1;
for (clog2=0; value>0; clog2=clog2+1)
value = value>>1;
end
endfunction
endmodule
```

# generate
> [Verilog语法：条件编译—Generate](https://blog.csdn.net/qq_26652069/article/details/90216879)

## generate if
两种形式，
```verilog
generate
  if(TSO_EN == 1) begin

  end
  else begin

  end
endgenerate
//or
generate
  if(TSO_EN == 1) begin

  end
endgenerate

generate
  if(TSO_EN == 0) begin

  end
endgenerate
```

## generate for
- 实现多个例化
- 实现多个assign
- 实现多个always

```verilog
generate for(i=0; i<4; i=i+1)					
  begin : gfor_block							
	assign temp[i] = indata[2*i+1:2*i];	
  end
endgenerate
```
等同于，
```verilog
assign temp[0] = indata[1:0];		
assign temp[1] = indata[3:2];		
assign temp[2] = indata[5:4];		
assign temp[3] = indata[7:6];
```

