---
layout: post
title: 一些 Verilog HDL 模块实例
date: 2018-8-8 10:48:30
category: 单片机&FPGA
draft: false
---

本文将会不定期更新一些常见的 Verilog HDL 模块实例供大家学习。如果对于数字电路不是很理解，或者对于语法不是很熟练，可以借鉴这里的例子，结合自己的项目进行模块开发。

FPGA 设计模式分为组合逻辑设计和时序逻辑设计，我们将这两类分开讨论。

<!--more-->


## 组合逻辑

### 8 位带进位端的加法器设计实例

```verilog
module adder_8(cout, sum, a, b, cin);
    output cout;
    output [7:0] sum;
    input cin;
    input [7:0] a, b;
    assign {cout, sum} = a + b + cin;
endmodule
```

### 指令译码电路设计实例（always）

```verilog
`define plus 3'd0
`define minus 3'd1
`define band 3'd2
`define bor 3'd3
`define unegate 3'd4

module alu(out, opcode, a, b);
output [7:0] out;
input [2:0] opcode;
input [7:0] a, b;
reg [7:0] out;

always @(opcode or a or b)
begin
    case(opcode)
        `plus: out=a+b;
        `minus: out=a-b;
        `band: out=a&b;
        `bor: out=a|b;
        `unegate: out=~a;
        default: out=8'hx;
    end case
end
endmodule
```

### 信号重组组合逻辑（task 和 always）
```verilog
module sort4(ra, rb, rc, rd, a, b, c, d);
parameter t=3;
output [t:0] ra, rb, rc, rd;
input [t:0] a, b, c, d;
reg [t:0] ra, rb, rc, rd;

always @(a or b or c or d)
begin: local //因为块有局部变量，块必须有名字
    reg[t:0] va, vb, vc, vd;
    {va, vb, vc, vd}={a, b, c, d};

    sort2(va,vc);
    sort2(vb,vd);
    sort2(va,vb);
    sort2(vc,vd);
    sort2(vb,vc);

    {ra, rb, rc, rd}={va, vb, vc, vd};
end

task sort2;
inout [t:0] x, y;
reg [t:0] tmp;
if(x>y)
    begin
        tmp=x;
        x=y;
        t=tmp;
    end
endtask
```

### 比较器的设计实例（利用赋值语句设计组合逻辑）
```verilog
module compare(equal, a, b);
parameter size=1;
output equal;
input [size-1:0] a, b;
    assign equal=(a==b)?1:0;
endmodule
```

### 3-8 译码器的实现（利用赋值语句设计组合逻辑）
```verilog
module decoder(out, in);
output [7:0] out;
input [2:0] in;
    assign out=1'b1<<in;
endmodule
```

### 多路器的设计实例
1. 利用赋值设计组合逻辑

```verilog
module emux1(out, a, b, sel);
    output out;
    input a, b, sel;
        assign out=sel?a:b;
endmodule
```

2. 利用 always case 设计组合逻辑

```verilog
module mux2(out, a, b, sel);
    output out;
    input a, b, sel;
    reg out;

    always @(a or b or sel)
    begin
        case(sel)
            1'b1: out=a;
            1'b0: out=b;
            default: out= 'bx;
        endcase
    end
endmodule
```

3. 利用 always if 设计组合逻辑

```verilog
module mux3(out, a, b, sel);
    output out;
    input a, b, sel;
    reg out;

    always @(a or b or sel)
    begin
        if(sel)
            out=a;
        else
            out=b;
    end
endmodule
```

### 奇偶校验位生成器设计实例
```verilog
module parity(even_numbits, odd_numbits, input_bus);
    output even_numbits, odd_numbits;
    input [7:0] input_bus;

    assign odd_numbits=^input_bus;
    //assign b=^a; equals assign b=a[0]^a[1]^a[2]^a[3]^a[4]^a[5]^a[6]^a[7];
    assign even_numbits=~odd_numbits;
endmodule
```

### 三态输出驱动器
1. 设计方案1。

```verilog
module trist1(out, in, enable);
    output out;
    input in, enable;
        assign out=enable?in:'bz;
endmodule
```

2. 设计方案2。

```verilog
module trist2(out, in, enable);
    output out;
    input in, enable;
        bufif1 mubuf1(out, in, enable);
endmodule
```

### 三态双向驱动器设计实例

```verilog
module bidir(tri_inout, out, in, en, b);
inout tri_out;
output out;
input in, en, b;
    assign tri_inout=en?in:'bz;
    assign out= tri_inout^b;
    //tri_inout 端口输出 in 信号，输入信号转交 out
endmodule
```
## 时序逻辑电路设计实例

### D 触发器设计实例
D 触发器目前正在逐渐替代已经淘汰的 JK 触发器，本设计以 D 触发器为设计对象。D 触发器在时钟上升沿同步输入输出。

```verilog
module dff(q, data, clk);
    output q;
    input data, clk;
    reg q;

    always @(posedge clk)
    begin
        q<=data;
    end
endmodule
```

### 电平敏感型锁存器设计实例
1. 设计方案1

```verilog
module latch1(q, data, clk);
    output q;
    input data, clk;
        assign q=clk?data:q;
endmodule
```

1. 设计方案2

```verilog
module latch1(q, data, clk);
    output q;
    input data, clk;
    reg q; //因为要用 always 块
    always @(clk or data)
    begin
        if(clk)
        q=data;
    end
endmodule
```

### 带置位和复位端口的电平敏感型锁存器设计实例

```verilog
module latch2(q, data, clk, set, reset);
    output q;
    input data, clk, set, reset;
        assign q=reset?0:(set?1:(clk?data:q));
endmodule
```

### 移位寄存器设计实例
```verilog
module shifter(din, clk, clr, dout);
    input din, clk, clr;
    output [7:0] dout;
    reg [7:0] dout;

    always @(posegde clk)
    begin
        if(clr)
            dout<=8'b0;
        else
            begin
            dout<=dout<<1;
            dout[0]<=din;
            end
    end
endmodule
```

### 8 位计数器的设计实例
1. 设计实例1

```verilog
module counter1(out, cout, data, load, cin, clk);
output [7:0] out;
output cout;
input [7:0] data;
input load, cin, clk;
reg [7:0] out;

always @(posedge clk)
    begin
        if(load)
            out<=data;
        else
            out<=out+cin;
    end

assign cout=(&out)&cin;
endmodule
```

2. 设计实例2

```verilog
module counter2(out, cout, data, load, cin, clk);
output [7:0] out;
output cout;
input [7:0] data;
input load, cin, clk;
reg [7:0] out;
reg cout;
reg [7:0] preout;

always @(posegde clk)
    begin
        out<=preout;
    end

always @(out or data or load or cin)
    begin
        {cout, preout}=out+cin;
        if(load)
            preout=data;
    end
endmodule
```
