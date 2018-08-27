# SoC
##System on chip with veliog

## 1. APB Bridge FSM
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
