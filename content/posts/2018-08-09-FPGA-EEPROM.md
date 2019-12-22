---
title: FPGA EEPROM Communication
date: 2018-08-09
draft: false
---

本文将会就 FPGA 的 I2C 总线实现进行实践，在测试的时候，我们使用了一个假象的 EEPROM 对象来进行测试，验证我们设计的合理性。

<!--more-->

## I2C 总线物理层与链路层
I2C 为两线双工总线，两根线分别为 SCL 和 SDA。空闲时 SCL 和 SDA 均为高电平。启动信号由 SCL 处于高电平，SDA 拉低开始。从这个时间后主机输出时钟 SCL，在 SCL 处于低电平状态下进行 SDA 改变，SCL 高电平情况下 SDA 数据被锁定，给丛机读取，最后 SCL 在高电平情况下拉高 SDA 完成传输。

每传输 8 个位，也就是一个字节后，下一个位的时钟期间，将SDA 释放，成为高电平，接收方将其拉低，应答完毕。

## EEPROM 测试模型
测试使用的 EEPROM 模型是使用 FPGA 实现的理想化简化行为模型，仅仅供测试使用。

```verilog
/* This file is made by EggyCat to implement AT24C02 */

`timescale 1ns/1ns
`define timeslice 100

module EEPROM(scl, sda);
input scl;
inout sda;
reg out_flag;

reg[7:0] memory[2047:0];
reg[10:0] address;
reg[7:0] memory_buf;
reg[7:0] sda_buf;
reg[7:0] shift;
reg[7:0] addr_byte;
reg[7:0] ctrl_byte;
reg[1:0] State;

integer i;

parameter r7=8'b10101111, w7=8'b10101110; //memory page 7
parameter r6=8'b10101101, w6=8'b10101100; //memory page 6
parameter r5=8'b10101011, w5=8'b10101010; //memory page 5
parameter r4=8'b10101001, w4=8'b10101000; //memory page 4
parameter r3=8'b10100111, w3=8'b10100110; //memory page 3
parameter r2=8'b10100101, w2=8'b10100100; //memory page 2
parameter r1=8'b10100011, w1=8'b10100010; //memory page 1
parameter r0=8'b10100001, w0=8'b10100000; //memory page 0

assign sda=(out_flag==1)?sda_buf[7]:1'bz; //sda whether output

initial
    begin
    addr_byte=0;
    ctrl_byte=0;
    out_flag=0;
    sda_buf=0;
    State=2'b00;
    memory_buf=0;
    address=0;
    shift=0;

    for(i=0;i<=2047;i=i+1)
        memory[i]=0;
    end

always @(negedge sda)
    if(scl==1)
        begin
            State=State+2'b01; //ModelSim6.0版本以上认为z到1是负跳变沿
            if(State==2'b11)
                disable write_to_eeprom;
        end

always @(posedge sda)
    if(scl==1)
        stop_W_R; //传输完毕
    else
        begin
            casex(State)
                2'b01: 
                begin
                    read_in;
                    if(ctrl_byte==w7 || ctrl_byte==w6 || ctrl_byte==w5 || ctrl_byte==w4 || ctrl_byte==w3 || ctrl_byte==w2 || ctrl_byte==w1 || ctrl_byte==w0)
                    begin
                        State=2'b10;
                        write_to_eeprom;
                    end
						  else
							State=2'b00;
					 end
                2'b11:
                read_from_eeprom;
                default:
                State=2'b00;
            endcase
        end

task stop_W_R;
    begin 
        State=2'b00;
        addr_byte=0;
        ctrl_byte=0;
        out_flag=0;
        sda_buf=0;
    end
endtask

task read_in;
    begin
        shift_in(ctrl_byte);
        shift_in(addr_byte);
    end
endtask

task write_to_eeprom;
    begin
        shift_in(memory_buf);
        address={ctrl_byte[3:1], addr_byte};
        memory[address]=memory_buf;
        $display("eeprom---memory[%0h]=%0h", address, memory[address]);
        State=2'b00;
    end
endtask

task read_from_eeprom;
    begin
        shift_in(ctrl_byte);
        if(ctrl_byte==r7 || ctrl_byte==r6 || ctrl_byte==r5 || ctrl_byte==r4 || ctrl_byte==r3 || ctrl_byte==r2 || ctrl_byte==r1 || ctrl_byte==r0)
            begin
                address={ctrl_byte[3:1], addr_byte};
                sda_buf=memory[address];
                shift_out;
                State=2'b00;
            end
    end
endtask

task shift_in;
    output[7:0]shift;
        begin
            @(posedge scl) shift[7]=sda;
            @(posedge scl) shift[6]=sda;
            @(posedge scl) shift[5]=sda;
            @(posedge scl) shift[4]=sda;
            @(posedge scl) shift[3]=sda;
            @(posedge scl) shift[2]=sda;
            @(posedge scl) shift[1]=sda;
            @(posedge scl) shift[0]=sda;
            @(negedge scl)
                begin
                    #`timeslice;
                    out_flag=1;
                    sda_buf=0;
                end
            @(negedge scl)
                #`timeslice out_flag=0;
        end
endtask

task shift_out;
    begin
        out_flag=1;
        for(i=6;i>=0;i=i-1)
            begin 
                @(negedge scl);
                #`timeslice;
                sda_buf=sda_buf<<1;
            end
        @(negedge scl) #`timeslice sda_buf[7]=1;
        @(negedge scl) #`timeslice out_flag=0;
    end
endtask
endmodule
```

## 模块分析
我们可以看到我们的 EEPROM 仿真模块是使用状态机来进行模拟的，`State` 为状态值，状态机闲置状态时 `State=2'b00`，EEPROM处于被写状态为 `State=2'b01`，EEPROM处于被读状态为 `State=2'b11`。

启动信号为 sda 的下降沿和 scl 处于高电平时，结束传输信号是 sda 处于上升沿且 scl 为高电平状态。在其他情况下，也就是 sda 处于上升沿且 scl 处于低电平状态时（因为 EEPROM 的第一个控制位一定为 1），我们就认定已经开始传输数据位了，这时触发 `read_in` 任务，开始顺序在 scl 的上升沿读取 sda 的数据到控制字节和数据字节中。

应答是通过一个连接标志 `out_flag` 实现的，sda 会在 `out_flag==1` 的情况下被连接在 sda_buf[7] 上面。应答是在最后的 scl 下降沿拉低 sda，我们在最后的下降沿将 sda_buf 置为 0，并将 `out_flag` 置 1 连接即可，在下一个 scl 下降沿（此时应答结束）将 `out_flag` 置 0，断开输出（sda 置为高阻）。

读任务完成后，EEPROM 获取了外部的沟通信息，信息分为两类，一类是外部请求读某地址的数据，另一类是外部请求写某地址的数据。我们接下来分析请求是读还是写，使用 if 语句比较第一个字节（控制字节）的信息，如果是请求写数据，那么触发写操作任务；相反如果控制字节是读信息，那么触发读操作任务。

写操作较为简单，因为不涉及应答，而读操作需要应答。

在读任务中，我们首先将需要发送的输出放置在 sda_buf 中，然后连接 sda
 和 sda_buf(`out_flag=1`)，因为我们在之前 read_in 的最后 scl 下降沿没有改变数据，也就是在一开始连接的时候就相当于已经发送出去了一位数据。为了避免冒险，我们定义了 `timeslice 延迟，在这里也可以使用非阻塞的方法避免冒险。在发送完第 8 位后，断开连接释放总线，等待主机拉高 sda（主机接收器不应答）结束通信。

在 Quartus Prime Lite 中编译提示，在 `task shift_in` 里面不能产生多事件序列，综合失败。

## 可综合的 EEPROM 读写器模型
程序实际上是一个嵌套的状态机，由主状态机和从状态机通过控制线启动的总线在不同的输入信号情况下构成不同功能的较复杂得到有限状态机（FSM）。

两个状态机的模型，分别为控制时序电路和开关组合电路，开关组合电路控制总线的开闭，有节奏地打开或者闭合，这样 SDA 可以按照 I2C 数据总线的格式输入和输出，使得 SDA 和 SCL 一起完成 EEPROM 的读写操作。

![][1]

其状态机的状态转移图如：

![][2]

状态机的状态编码采用独热编码，减少实现状态机的组合逻辑数目，减少复杂性，提高系统的速度，通常用于高速状态机中（也有编码输出型状态机）。

```verilog
`timescale 1ns/1ns

module EEPROM_WR(SDA, SCL, ACK, RESET, CLK, WR, RD, ADDR, DATA);
output SCL;
output ACK;
input RESET;
input CLK;
input WR, RD;
input [10:0] ADDR;
inout SDA;
inout [7:0] DATA;

reg ACK;
reg SCL;
reg WF, RF;
reg FF;
reg [1:0] head_buf;
reg [1:0] stop_buf;
reg [7:0] sh8out_buf;
reg [8:0] sh8out_state;
reg [9:0] sh8in_state;
reg [1:0] head_state;
reg [2:0] stop_state;
reg [10:0] main_state;
reg [7:0] data_from_rm;
reg link_sda;
reg link_read;
reg link_head;
reg link_write;
reg link_stop;
wire sda1, sda2, sda3, sda4;

/* Switch Signal */
assign sda1=(link_head)? head_buf[1] : 1'b0;
assign sda2=(link_write)? sh8out_buf[7] : 1'b0;
assign sda3=(link_stop)? stop_buf[1] : 1'b0;
assign sda4=(sda1|sda2|sda3);
assign SDA=(link_sda)? sda4: 1'bz;
assign DATA=(link_read)? data_from_rm : 8'hzz;

/* Main FSM State Definition */
parameter
    IDLE        = 11'b00000000001,
    Ready       = 11'b00000000010,
    Write_start = 11'b00000000100,
    Ctrl_write  = 11'b00000001000,
    Addr_write  = 11'b00000010000,
    Data_write  = 11'b00000100000,
    Read_start  = 11'b00001000000,
    Ctrl_read   = 11'b00010000000,
    Data_read   = 11'b00100000000,
    Stop        = 11'b01000000000,
    Ackn        = 11'b10000000000,

/* Parallel to Serial Output State Definition */
    sh8out_bit7 = 9'b000000001,
    sh8out_bit6 = 9'b000000010,
    sh8out_bit5 = 9'b000000100,
    sh8out_bit4 = 9'b000001000,
    sh8out_bit3 = 9'b000010000,
    sh8out_bit2 = 9'b000100000,
    sh8out_bit1 = 9'b001000000,
    sh8out_bit0 = 9'b010000000,
    sh8out_end  = 9'b100000000,

/* Serial to Parallel Output State Definition */
    sh8in_begin = 10'b0000000001,
    sh8in_bit7  = 10'b0000000010,
    sh8in_bit6  = 10'b0000000100,
    sh8in_bit5  = 10'b0000001000,
    sh8in_bit4  = 10'b0000010000,
    sh8in_bit3  = 10'b0000100000,
    sh8in_bit2  = 10'b0001000000,
    sh8in_bit1  = 10'b0010000000,
    sh8in_bit0  = 10'b0100000000,
    sh8in_end   = 10'b1000000000,

/* Start of Transmission */
    head_begin = 2'b01,
    head_bit   = 2'b10,

/* End of Transmission */
    stop_begin = 3'b001,
    stop_bit   = 3'b010,
    stop_end   = 3'b100,

    YES = 1'b1,
    NO  = 1'b0;

/* Generate SCL (CLK divided by 2) */
always @(negedge CLK)
    if(RESET)
        SCL<=0;
    else
        SCL<=~SCL;

/* Main FSM Status */
always @(posedge CLK)
    if(RESET) // RESET
        begin
            link_read  <= NO;
            link_write <= NO;
            link_head  <= NO;
            link_stop  <= NO;
            link_sda   <= NO;
            ACK        <= 0;
            RF         <= 0;
            WF         <= 0;
            FF         <= 0;
            main_state <= IDLE;
        end
    else
        begin
            casex(main_state)
                IDLE:
                    begin
                        link_read  <= NO;
                        link_write <= NO;
                        link_head  <= NO;
                        link_stop  <= NO;
                        link_sda   <= NO;
                        if(WR)
                            begin
                                WF<=1;
                                main_state<=Ready;
                            end
                        else if(RD)
                            begin
                                RF<=1;
                                main_state<=Ready;
                            end
                        else
                            begin
                                WF<=0;
                                RF<=0;
                                main_state<=IDLE;
                            end
                    end

                Ready:
                    begin
                        link_read  <= NO;
                        link_write <= NO;
                        link_head  <= YES;
                        link_stop  <= NO;
                        link_sda   <= YES;
                        head_buf[1:0] <= 2'b10;
                        stop_buf[1:0] <= 2'b01;
                        head_state    <= head_begin;
                        FF  <= 0;
                        ACK <= 0;
                        main_state <= Write_start;
                    end

                Write_start:
                    if(FF==0)
                        shift_head;
                    else
                        begin
                            sh8out_buf[7:0] <={1'b1, 1'b0, 1'b1, 1'b0, ADDR[10:8], 1'b0};
                            link_head <= NO;
                            link_write <= YES;
                            FF <= 0;
                            sh8out_state <= sh8out_bit6;
                            main_state <= Ctrl_write;
                        end

                Ctrl_write:
                    if(FF==0)
                        shift8_out;
                    else
                        begin
                            FF <= 0;
                            if(WF)
                                begin
                                    sh8out_state <= sh8out_bit7;
                                    sh8out_buf[7:0] <= DATA;
                                    main_state <= Data_write;
                                end
                            if(RF)
                                begin
                                    head_buf <= 2'b10;
                                    head_state <= head_begin;
                                    main_state <= Read_start;
                                end
                        end

                Addr_write:
                    if(FF==0)
                        shift8_out;
                    else
                        begin
                            FF <= 0;
                            if(WF)
                                begin 
                                    sh8out_state <= sh8out_bit7;
                                    sh8out_buf[7:0] <= DATA;
                                    main_state <= Data_write;
                                end
                            if(RF)
                                begin
                                    head_buf <= 2'b10;
                                    head_state <= head_begin;
                                    main_state <= Read_start;
                                end
                        end

                Data_write:
                    if(FF==0)
                        shift8_out;
                    else
                        begin
                            stop_state <= stop_begin;
                            main_state <= Stop;
                            link_write <= NO;
                            FF <= 0;
                        end
                
                Read_start:
                    if(FF==0)
                        shift_head;
                    else
                        begin
                            sh8out_buf <= {1'b1, 1'b0, 1'b1, 1'b0, ADDR[10:8], 1'b1};
                            link_head <= NO;
                            link_sda <= YES;
                            link_write <= YES;
                            FF <= 0;
                            sh8out_state <= sh8out_bit6;
                            main_state <= Ctrl_read;
                        end

                Data_read:
                    if(FF==0)
                        shift8in;
                    else
                        begin
                            link_stop <= YES;
                            link_sda <= YES;
                            stop_state <= stop_bit;
                            FF <= 0;
                            main_state <= Stop;
                        end
                
                Stop:
                    if(FF==0)
                        shift_stop;
                    else
                        begin
                            ACK <= 1;
                            FF <= 0;
                            main_state <= Ackn;
                        end

                Ackn:
                    begin
                        ACK <= 0;
                        WF <= 0;
                        RF <= 0;
                        main_state <= IDLE;
                    end

                default: 
                    main_state <= IDLE;
            endcase
        end

/* Serial to Parallel Task */
task shift8in;
begin
    casex(sh8in_state)
        sh8in_begin:
            sh8in_state <= sh8in_bit7;
        sh8in_bit7:
            if(SCL)
                begin
                    data_from_rm[7] <= SDA;
                    sh8in_state <= sh8in_bit6;
                end
            else
                sh8in_state <= sh8in_bit7;
        sh8in_bit6:
            if(SCL)
                begin
                    data_from_rm[6] <= SDA;
                    sh8in_state <= sh8in_bit5;
                end
            else
                sh8in_state <= sh8in_bit6;
        sh8in_bit5:
            if(SCL)
                begin
                    data_from_rm[5] <= SDA;
                    sh8in_state <= sh8in_bit4;
                end
            else
                sh8in_state <= sh8in_bit5;
        sh8in_bit4:
            if(SCL)
                begin
                    data_from_rm[4] <= SDA;
                    sh8in_state <= sh8in_bit3;
                end
            else
                sh8in_state <= sh8in_bit4;
        sh8in_bit3:
            if(SCL)
                begin
                    data_from_rm[3] <= SDA;
                    sh8in_state <= sh8in_bit2;
                end
            else
                sh8in_state <= sh8in_bit3;
        sh8in_bit2:
            if(SCL)
                begin
                    data_from_rm[2] <= SDA;
                    sh8in_state <= sh8in_bit1;
                end
            else
                sh8in_state <= sh8in_bit2;
        sh8in_bit1:
            if(SCL)
                begin
                    data_from_rm[1] <= SDA;
                    sh8in_state <= sh8in_bit0;
                end
            else
                sh8in_state <= sh8in_bit1;
        sh8in_bit0:
            if(SCL)
                begin
                    data_from_rm[0] <= SDA;
                    sh8in_state <= sh8in_end;
                end
            else
                sh8in_state <= sh8in_bit0;
        sh8in_end:
            if(SCL)
                begin
                    link_read <= YES;
                    FF <= 1;
                    sh8in_state <= sh8in_bit7;
                end
            else
                sh8in_state <= sh8in_end;
        default:
            begin
                link_read <= NO;
                sh8in_state <= sh8in_bit7;
            end
    endcase
end
endtask

/* Parallel to Serial Task */
task shift8_out;
begin
    casex(sh8out_state)
        sh8out_bit7:
            if(!SCL)
                begin
                    link_sda <= YES;
                    link_write <= YES;
                    sh8out_state <= sh8out_bit6;
                end
            else
                sh8out_state <= sh8out_bit7;
        sh8out_bit6:
            if(!SCL)
                begin
                    link_sda <= YES;
                    link_write <= YES;
                    sh8out_state <= sh8out_bit5;
                    sh8out_buf <= sh8out_buf<<1;
                end
            else
                sh8out_state <= sh8out_bit6;
        sh8out_bit5:
            if(!SCL)
                begin
                    sh8out_state <= sh8out_bit4;
                    sh8out_buf <= sh8out_buf<<1;
                end
            else
                sh8out_state <= sh8out_bit5;
        sh8out_bit4:
            if(!SCL)
                begin
                    sh8out_state <= sh8out_bit3;
                    sh8out_buf <= sh8out_buf<<1;
                end
            else
                sh8out_state <= sh8out_bit4;
        sh8out_bit3:
            if(!SCL)
                begin
                    sh8out_state <= sh8out_bit2;
                    sh8out_buf <= sh8out_buf<<1;
                end
            else
                sh8out_state <= sh8out_bit3;
        sh8out_bit2:
            if(!SCL)
                begin
                    sh8out_state <= sh8out_bit1;
                    sh8out_buf <= sh8out_buf<<1;
                end
            else
                sh8out_state <= sh8out_bit2;
        sh8out_bit1:
            if(!SCL)
                begin
                    sh8out_state <= sh8out_bit0;
                    sh8out_buf <= sh8out_buf<<1;
                end
            else
                sh8out_state <= sh8out_bit1;
        sh8out_bit0:
            if(!SCL)
                begin
                    sh8out_state <= sh8out_end;
                    sh8out_buf <= sh8out_buf<<1;
                end
            else
                sh8out_state <= sh8out_bit0;
        sh8out_end:
            if(!SCL)
                begin
                    link_sda <= NO;
                    link_write <= NO;
                    FF <= 1;
                end
            else
                sh8out_state <= sh8out_end;
    endcase
end
endtask

/* Start of Transmission Task */
task shift_head;
begin
    casex(head_state)
        head_begin:
            if(!SCL)
                begin
                    link_write <= 0;
                    link_sda <= YES;
                    link_head <= YES;
                    head_state <= head_bit;
                end
            else
                head_state <= head_begin;
        head_bit:
            if(SCL)
                begin
                    FF <= 1;
                    head_buf <= head_buf << 1;
                    head_state <= head_bit;
                end
            else
                head_state <= head_bit;
    endcase
end
endtask

/* End of Transmission */
task shift_stop;
begin
    casex(stop_state)
        stop_begin:
            if(!SCL)
                begin
                    link_sda <= YES;
                    link_write <= NO;
                    link_stop <= YES;
                    stop_state <= stop_bit;
                end
            else
                stop_state <= stop_begin;
        stop_bit:
            if(SCL)
                begin
                    stop_buf <= stop_buf << 1;
                    stop_state <= stop_end;
                end
            else
                stop_state <= stop_bit;
        stop_end:
            if(!SCL)
                begin
                    link_head <= NO;
                    link_stop <= NO;
                    link_sda <= NO;
                    FF <= 1;
                end
            else
                stop_state <= stop_end;
    endcase
end
endtask
endmodule
```

## 读写器模型解析
在这个读写器模型中，我认为难点在于两个，一个是开关的控制，另外一个是状态机状态的转换过程。

首先我们来讨论开关的控制。我们用了四个开关来控制 SDA 线的输入和输出，其中 `link_sda` 为总的使能开关，控制着总线的释放还是占据，释放的时候，sda 处于高阻态，也就是处于读取状态。而剩余的三个开关 `link_head`/`link_write`/`link_stop` 仅仅控制 sda 的移位寄存器连接关系，而这三个开关是互斥的，也就是同时 sda 所连接的寄存器只可能是 “启动信号”，“数据信号”，“停止信号” 中的一种寄存器中的最高位。

之所以启动信号和停止信号都是两位的寄存器，是因为我们要通过 **位移寄存器** 来产生跳边沿。而数据寄存器没有多于一位，我们在描述中`shift8_out` 任务中，8 位的数据只位移了 7 次，第一次只连接寄存器，而不位移。

在发送完启动信号之后的 `Write_start` 状态中，我们的 `shift8_out` 是从第 bit6 状态开始的，因为在这之前已经完成了开关，相当于已经发送了一位了。而在 `Addr_write` 中，我们则是从 bit7 状态开始的，因为我们其实在这个时候空出了一个 SCL 的低电平总线没有占据，在这个时间内是应答信号的时间。

## 状态机分析和 RTL 网表

状态机的分析很清晰：

![][3]

相比之下，RTL 网表的分析就有些复杂了，因为我们设计的状态机用到了许多寄存器：

![][4]

至此，我们完成了对于 EEPROM 读写器的模型建立过程，在未来我们将会对于这个模型使用我们写好的 EEPROM 模块进行测试。

[1]: /img/2018-8-9-FPGA-EEPROM/struct.png
[2]: /img/2018-8-9-FPGA-EEPROM/FSM.png
[3]: /img/2018-8-9-FPGA-EEPROM/Serial_Check_main_state.png
[4]: /img/2018-8-9-FPGA-EEPROM/RTL.png
