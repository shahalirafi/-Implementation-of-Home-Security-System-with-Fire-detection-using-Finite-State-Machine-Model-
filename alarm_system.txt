module alarm_system(
    input clk,
    input reset, // Added reset input
    input start_button,
    input [3:0] password,
    input smoke_detected,
    input fire_detected,
    input high_temp_detected,
    output reg alarm_active,
    output reg alarm_silent,
    output reg call_fire_service
);

    // Define state parameters
    localparam [2:0] 
        IDLE = 3'b000, 
        ENTER_PASS = 3'b001, 
        ALARM_ACTIVE = 3'b010, 
        ALARM_SILENT = 3'b011, 
        SMOKE_DETECTOR = 3'b100, 
        CALL_FIRE_SERVICE = 3'b101;

    // Internal state variable
    reg [2:0] state;
    reg [2:0] next_state;

    // FSM state transition with asynchronous reset
    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= IDLE;
        else
            state <= next_state;
    end

    // Next state logic
    always @(*) begin
        // Default outputs and next state
        alarm_active = 0;
        alarm_silent = 0;
        call_fire_service = 0;
        next_state = state;

        case (state)
            IDLE: begin
                if (start_button) 
                    next_state = ENTER_PASS;
                else 
                    next_state = IDLE;
            end
            
            ENTER_PASS: begin
                if (password == 4'b1111) // Assume 4'b1111 is the correct password
                    next_state = ALARM_SILENT;
                else
                    next_state = ALARM_ACTIVE;
            end

            ALARM_ACTIVE: begin
                alarm_active = 1;
                if (password == 4'b1111)
                    next_state = ALARM_SILENT;
                else if (fire_detected)
                    next_state = CALL_FIRE_SERVICE;
            end
            
            ALARM_SILENT: begin
                alarm_silent = 1;
                if (smoke_detected || high_temp_detected)
                    next_state = SMOKE_DETECTOR;
                else if (!start_button)
                    next_state = IDLE;
            end
            
            SMOKE_DETECTOR: begin
                if (fire_detected)
                    next_state = CALL_FIRE_SERVICE;
                else if (!smoke_detected && !high_temp_detected)
                    next_state = ALARM_SILENT;
            end

            CALL_FIRE_SERVICE: begin
                call_fire_service = 1;
                if (!fire_detected)
                    next_state = ALARM_SILENT;
            end

            default: begin
                next_state = IDLE;
            end
        endcase
    end
endmodule
