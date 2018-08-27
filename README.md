# SoC
## System on chip with veliog

## 1. APB Bridge FSM
![alt text](https://github.com/chaeyoonlee/SoC/blob/master/ARP%20bridge.png)
<pre><code>
//Instead of reset=0,IDLE=1,SETUP=2,ENABLE=3,..
//do this when it's low power
`define IDLE 3`b001
`define SETUP 3`b010
`define ENABLE 3`b100

// NEXT STATE 계산부 
always@(state or inputs or ...) 
begin
case (state)
          'IDLE: if(transfer) 
			nxt_st = SETUP;
			PSELx=0;
			PENABLE = 0;
		else
             		nxt_st = IDLE;
          'SETUP: 
             		nxt_st = ENABLE;
			PSELx=1;
			PENABLE = 0;

          'ENABLE=: if(transfer)
             		nxt_st = IDLE;
			PSELx=1;
			PENABLE = 1;
              	else
             		nxt_st = SETUP;
         
          default : nxt_st = IDLE;          
        endcase




end

// state register transition
always@(posedge clk) begin   
	state<=next_state; 
end  

// OUTPUT calculation
assign PSELx = (state == 'SETUP || state == 'ENABLE)
// Mealy 

</code></pre>

## 2. 8:1 MUX
![alt text](https://github.com/chaeyoonlee/SoC/blob/master/8_1_MUX.png)
<pre><code>
module mux(out, i0, i1, i2, i3, i4, i5, i6, i7, s2, s1, s0);
    
      output out;
    input i0, i1, i2, i3, i4, i5, i6, i7;
    input s2, s1, s0;
    // output type as reg
    reg out;
    always @(s2 or s1 or s0 or i0 or i1 or i2 or i3 or i4 or i5 or i6 or i7)
    begin
    case ({s2, s1, s0})
    3'b000: out = i0; 
    3'b001: out = i1;
    3'b010: out = i2;
    3'b011: out = i3;
    3'b100: out = i4; 
    3'b101: out = i5;
    3'b110: out = i6;
    3'b111: out = i7;
    
    default: out = 1'bx;
    endcase
    end

endmodule

module stimulus;
//  입력으로 연결되는 변수들을 정의 
reg IN0, IN1, IN2, IN3,  IN4, IN5, IN6, IN7; 
reg S0, S1, S2;
// 출력 wire의 선언 
wire OUTPUT;
// 멀티플렉스의 파생 
mux mymux (OUTPUT, IN0, IN1, IN2, IN3, IN4, IN5, IN6, IN7, S0, S1, S2);
// 스티뮬러스의 입력 
initial 
begin 
// 입력 라인을 셋 
IN0 = 1;   IN1 = 0;   IN2 = 1;   IN3 = 0; IN4 = 1; IN5 = 0; IN6 = 1; IN7 = 0;
#1 $display ("IN0 = %b, IN1 = %b, IN2 = %b, IN3 = %b, IN4 = %b\n, IN5 = %b\n, IN6 = %b\n,IN7 = %b", IN0, IN1, IN2, IN3,IN4, IN5, IN6, IN7); 


// IN0 을 선택 


S2 = 0; S1 = 0; S0 = 0; 
#1 $display ("S2 = %b, S1 = %b, S0 = %b, OUTPUT = %b \n", S2, S1, S0, OUTPUT);

S2 = 0; S1 = 0; S0 = 1; 
#1 $display ("S2 = %b, S1 = %b, S0 = %b, OUTPUT = %b \n", S2, S1, S0, OUTPUT);

S2 = 0; S1 = 1; S0 = 0; 
#1 $display ("S2 = %b, S1 = %b, S0 = %b, OUTPUT = %b \n", S2, S1, S0, OUTPUT);

S2 = 0; S1 = 1; S0 = 1; 
#1 $display ("S2 = %b, S1 = %b, S0 = %b, OUTPUT = %b \n", S2, S1, S0, OUTPUT);

S2 = 1; S1 = 0; S0 = 0; 
#1 $display ("S2 = %b, S1 = %b, S0 = %b, OUTPUT = %b \n", S2, S1, S0, OUTPUT);

S2 = 1; S1 = 0; S0 = 1; 
#1 $display ("S2 = %b, S1 = %b, S0 = %b, OUTPUT = %b \n", S2, S1, S0, OUTPUT);

S2 = 1; S1 = 1; S0 = 0; 
#1 $display ("S2 = %b, S1 = %b, S0 = %b, OUTPUT = %b \n", S2, S1, S0, OUTPUT);

S2 = 1; S1 = 1; S0 = 1; 
#1 $display ("S2 = %b, S1 = %b, S0 = %b, OUTPUT = %b \n", S2, S1, S0, OUTPUT);

end
endmodule

</code></pre>

## 3. 8 bits adder
![alt text](https://github.com/chaeyoonlee/SoC/blob/master/8%20bits%20adder.png)
<pre><code>
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date: 2017/09/19 23:03:59
// Design Name: 
// Module Name: 8bitadder
// Project Name: 
// Target Devices: 
// Tool Versions: 
// Description: 
// 
// Dependencies: 
// 
// Revision:
// Revision 0.01 - File Created
// Additional Comments:
// 
//////////////////////////////////////////////////////////////////////////////////


module bitadder(a, b, cin, sum, cout);
input [07:0] a;
input [07:0] b;
input cin;
output [7:0]sum;
output cout;
wire[6:0] c;
add a1(a[0],b[0],cin,sum[0],c[0]);
add a2(a[1],b[1],c[0],sum[1],c[1]);
add a3(a[2],b[2],c[1],sum[2],c[2]);
add a4(a[3],b[3],c[2],sum[3],c[3]);
add a5(a[4],b[4],c[3],sum[4],c[4]);
add a6(a[5],b[5],c[4],sum[5],c[5]);
add a7(a[6],b[6],c[5],sum[6],c[6]);
add a8(a[7],b[7],c[6],sum[7],cout);
endmodule

module add(a, b, cin, sum, cout);
input a;
input b;
input cin;
output sum;
output cout;
assign sum=(a^b^cin);
assign cout=((a&b)|(b&cin)|(a&cin));
endmodule


//test
module test;
reg [7:0] a;
reg [7:0] b;
reg cin;
wire [7:0] sum;
wire cout;
bitadder uut (.a(a),.b(b),.cin(cin),.sum(sum),.cout(cout) );
initial 
begin
a=1; b=1; cin=0;
#5 $display("a = %b, b = %b, cin = %b, cout = %b",a,b,cin, cout);
a=0; b=1; cin=0;
#5 $display("a = %b, b = %b, cin = %b, cout = %b",a,b,cin, cout);
a=2; b=1; cin=0;
#5 $display("a = %b, b = %b, cin = %b, cout = %b",a,b,cin, cout);
a=2; b=4; cin=0;
#5 $display("a = %b, b = %b, cin = %b, cout = %b",a,b,cin, cout);

end
endmodule

</code></pre>

## 4. FSM
![alt text](https://github.com/chaeyoonlee/SoC/blob/master/FSM.png)
<pre><code>
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
module ser(clk, x, z );    
    input clk, x; output z;
    reg [2:0] state, nxt_st; 
    parameter S0=0,S1=1,S2=2,S3=3;
    
    always @(posedge clk) 
    begin
    state<=nxt_st; 
    end
    
    assign z = (state==S3);
    
    always @(state or x)
    begin
        case (state)
          S0: if(x)
             nxt_st = S1;
              else
             nxt_st = S0;
          S1: if(x)
             nxt_st = S3;
              else
             nxt_st = S2;
          S2: if(x)
             nxt_st = S2;
              else
             nxt_st = S3;
          S3 : nxt_st = S0;
          default : nxt_st = S0;          
        endcase
    end
   endmodule

module test;
    reg clk;
    reg x;
    wire z;
    ser fsm(.clk(clk), .x(x), .z(z));
    
    initial
        clk = 0;
        always
        #5 clk = ~clk;    
    initial begin  
    #5 x = 0;
    $monitor("x = %d, output z = %d", x, z);
    #10 x = 1;
    $monitor("x = %d, output z = %d", x, z);
    #15 x = 0;
    $monitor("x = %d, output z = %d", x, z);
    #20 x = 1;
    $monitor("x = %d, output z = %d", x, z);
    end
endmodule

</code></pre>

