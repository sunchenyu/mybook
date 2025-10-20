# sgip.lua脚本

```lua
-- SGIP.lua
-- SGIP protocol
-- author: h_Davy
do
    local p_SGIP = Proto("SGIP","SGIP","SGIP Protocol")
    local f_Length = ProtoField.uint32("SGIP.length","Message Length",base.DEC)
    local f_CommandId = ProtoField.uint32("SGIP.commandId","Command ID",base.HEX,{[1]="Bind",[0x80000001]="BindResp",[2]="UnBind",[3]="Submit",[0x80000003]="SubmitResp",[4]="Deliver",[0x80000004]="DeliverResp",[5]="Report",[0x80000005]="ReportResp"})
    local f_NodeId = ProtoField.uint32("SGIP.NodeId","NodeId",base.DEC);
    local f_Mmddhhmmss = ProtoField.uint32("SGIP.mmddhhmmss","Mmddhhmmss",base.DEC);
    local f_SequenceId = ProtoField.uint32("SGIP.sequenceId","Sequence ID",base.DEC);
    local f_Data = ProtoField.bytes("SGIP.Data","Data")
    local f_LoginType = ProtoField.uint8("SGIP.loginType","Login Type",base.HEX)
    local f_LoginName = ProtoField.string("SGIP.loginName","Login Name")
    local f_LoginPass = ProtoField.string("SGIP.loginPass","Login Pass")
    local f_Result = ProtoField.uint8("SGIP.result","Result",base.DEC,{[0]="OK"})
    local f_Text = ProtoField.string("SGIP.text")
    local f_SPNumber = ProtoField.string("SGIP.SPNumber","SP Number")
    local f_ChargeNumber = ProtoField.string("SGIP.ChargeNumber","Charge Number")
    local f_UserCount = ProtoField.uint8("SGIP.userCount","User Count",base.DEC)
    local f_UserNumber = ProtoField.string("SGIP.UserNumber","User Number")
    local f_CorpId = ProtoField.string("SGIP.CorpId","CorpId")
    local f_ServiceType = ProtoField.string("SGIP.ServiceType","Service Type")
    local f_FeeType = ProtoField.uint8("SGIP.FeeType","Fee Type")
    local f_FeeValue = ProtoField.string("SGIP.FeeValue","Fee Value")
    local f_GivenValue = ProtoField.string("SGIP.GivenValue","Given Value")
    local f_AgentFlag = ProtoField.uint8("SGIP.AgentFlag","Agent Flag")
    local f_MorelatetoMTFlag = ProtoField.uint8("SGIP.MorelatetoMTFlag","Morelateto MT Flag")
    local f_Priority = ProtoField.uint8("SGIP.Priority","Priority")
    local f_ExpireTime = ProtoField.string("SGIP.ExpireTime","ExpireTime")
    local f_ScheduleTime = ProtoField.string("SGIP.ScheduleTime","ScheduleTime")
    local f_ReportFlag = ProtoField.uint8("SGIP.ReportFlag","ReportFlag")
    local f_TP_pid = ProtoField.uint8("SGIP.TP_pid","TP_pid")
    local f_TP_udhi = ProtoField.uint8("SGIP.TP_udhi","TP_udhi")
    local f_MessageCoding = ProtoField.uint8("SGIP.MessageCoding","MessageCoding")
    local f_MessageType = ProtoField.uint8("SGIP.MessageType","MessageType")
    local f_MessageLength = ProtoField.uint8("SGIP.MessageLength","MessageLength")
    local f_MessageContent = ProtoField.string("SGIP.MessageContent","MessageContent")
    local f_SubmitSequenceNumber = ProtoField.uint32("SGIP.SubmitSequenceNumber","SubmitSequenceNumber")
    local f_ReportType = ProtoField.uint8("SGIP.ReportType","ReportType")
    --local f_UserNumber = ProtoField.string("SGIP.UserNumber","UserNumber")
    local f_State = ProtoField.uint8("SGIP.State","State")
    local f_ErrorCode = ProtoField.uint8("SGIP.ErrorCode","ErrorCode")
    --local f_TP_udhi = ProtoField.uint8("SGIP.TP_udhi","TP_udhi")
    
    p_SGIP.fields = {f_Length,f_CommandId,f_NodeId,f_SequenceId,f_Mmddhhmmss,f_Data,f_LoginType,f_LoginName
    ,f_LoginPass,f_Result,f_Text,f_UserCount,f_SPNumber,f_ChargeNumber,f_UserNumber
    ,f_CorpId,f_ServiceType,f_FeeType,f_FeeValue,f_GivenValue,f_AgentFlag,f_MorelatetoMTFlag
    ,f_Priority,f_ExpireTime,f_ScheduleTime,f_ReportFlag,f_TP_pid,f_TP_udhi,f_MessageCoding
    ,f_MessageType,f_MessageLength,f_MessageContent,f_SubmitSequenceNumber,f_ReportType
    ,f_State,f_ErrorCode}
    
    local data_dis = Dissector.get("data")
    --
    local function SGIP_Bind(buf,pkt,t)
     pkt.cols.info:prepend("[SGIP_Bind]" .. " ")
     --
     t:add(f_LoginType,buf(20,1))
     t:add(f_LoginName,buf(21,16))
     t:add(f_LoginPass,buf(37,16))
    end
    --
    local function SGIP_Submit(buf,pkt,t)
     pkt.cols.info:prepend("[SGIP_Submit]" .. " ")
     t:add(f_SPNumber,buf(20,21))
     t:add(f_ChargeNumber,buf(41,21))
     local v_UserCount = buf(62,1)
     t:add(f_UserCount,v_UserCount)
     v_UserCount = v_UserCount:uint()
     -- 目标号码
     local v_curPos = 63
     for i=1,v_UserCount do
      t:add(f_UserNumber,buf(v_curPos,21))
      v_curPos = v_curPos+21
     end
     t:add(f_CorpId,buf(v_curPos,5))
     v_curPos=v_curPos+5
     t:add(f_ServiceType,buf(v_curPos,10))
     v_curPos=v_curPos+10
     t:add(f_FeeType,buf(v_curPos,1))
     v_curPos=v_curPos+1
     t:add(f_FeeValue,buf(v_curPos,6))
     v_curPos=v_curPos+6
     t:add(f_GivenValue,buf(v_curPos,6))
     v_curPos=v_curPos+6
     t:add(f_AgentFlag,buf(v_curPos,1))
     v_curPos=v_curPos+1
     t:add(f_MorelatetoMTFlag,buf(v_curPos,1))
     v_curPos=v_curPos+1
     t:add(f_Priority,buf(v_curPos,1))
     v_curPos=v_curPos+1
     t:add(f_ExpireTime,buf(v_curPos,16))
     v_curPos=v_curPos+16
     t:add(f_ScheduleTime,buf(v_curPos,16))
     v_curPos=v_curPos+16
     t:add(f_ReportFlag,buf(v_curPos,1))
     v_curPos=v_curPos+1
     t:add(f_TP_pid,buf(v_curPos,1))
     v_curPos=v_curPos+1
     t:add(f_TP_udhi,buf(v_curPos,1))
     v_curPos=v_curPos+1
     local v_MessageCoding=buf(v_curPos,1)
     t:add(f_MessageCoding,v_MessageCoding)
     v_MessageCoding = v_MessageCoding:uint()
     v_curPos=v_curPos+1
     t:add(f_MessageType,buf(v_curPos,1))
     v_curPos=v_curPos+1
     local v_MessageLength=buf(v_curPos,4)
     t:add(f_MessageLength,v_MessageLength)
     v_MessageLength=v_MessageLength:uint()
     v_curPos=v_curPos+4
     t:add(f_MessageContent,buf(v_curPos,v_MessageLength))
    end
    --
    local function SGIP_Deliver(buf,pkt,t)
     pkt.cols.info:prepend("[SGIP_Deliver]" .. " ")
     t:add(f_UserNumber,buf(20,21))
     t:add(f_SPNumber,buf(41,21))
     t:add(f_TP_pid,buf(62,1))
     t:add(f_TP_udhi,buf(63,1))
     local v_MessageCoding=buf(64,1)
     t:add(f_MessageCoding,v_MessageCoding)
     v_MessageCoding=v_MessageCoding:uint()
     local v_MessageLength=buf(65,4)
     t:add(f_MessageLength,v_MessageLength)
     v_MessageLength=v_MessageLength:uint()
     t:add(f_MessageContent,buf(69,v_MessageLength))
    end
    --
    local function SGIP_Report(buf,pkt,t)
     pkt.cols.info:prepend("[SGIP_Report]" .. " ")
     --t:add(f_SubmitSequenceNumber,buf(20,12))
     t:add(f_SubmitSequenceNumber,buf(28,4))
     t:add(f_ReportType,buf(32,1))
     t:add(f_UserNumber,buf(33,21))
     t:add(f_State,buf(54,1))
     t:add(f_ErrorCode,buf(55,1))
    end
    --
    local function SGIP_Response(buf,pkt,t)
     t:add(f_Result,buf(20,1))
    end
    local function SGIP_Bind_Response(buf,pkt,t)
     pkt.cols.info:prepend("[SGIP_BindResp]" .. " ")
     t:add(f_Result,buf(20,1))
    end
    local function SGIP_SubmitResponse(buf,pkt,t)
     pkt.cols.info:prepend("[SGIP_SubmitResp]" .. " ")
     t:add(f_Result,buf(20,1))
    end
    local function SGIP_DeliverResponse(buf,pkt,t)
     pkt.cols.info:prepend("[SGIP_DeliverResp]" .. " ")
     t:add(f_Result,buf(20,1))
    end
    local function SGIP_ReportResponse(buf,pkt,t)
     pkt.cols.info:prepend("[SGIP_ReportResp]" .. " ")
     t:add(f_Result,buf(20,1))
    end
    --
    local function SGIP_dissector(buf,pkt,root)
     local buf_len = buf:len();
     if buf_len < 8 then return false end
     local v_length = buf(0,4)
     local v_command = buf(4,4)
     local v_nodeId = buf(8,4)
     local v_mmddhhmmss = buf(12,4)

     local time_parsed = "无效时间"
     if v_mmddhhmmss and v_mmddhhmmss:uint() then
        local num = v_mmddhhmmss:uint()
        local time_str = string.format("%010d", num)

        if #time_str == 10 then
            local mm = tonumber(time_str:sub(1, 2))
            local dd = tonumber(time_str:sub(3, 4))
            local hh = tonumber(time_str:sub(5, 6))
            local mi = tonumber(time_str:sub(7, 8))
            local ss = tonumber(time_str:sub(9,10))

            if mm and dd and hh and mi and ss and
               mm >= 1 and mm <= 12 and
               dd >= 1 and dd <= 31 and
               hh >= 0 and hh <= 23 and
               mi >= 0 and mi <= 59 and
               ss >= 0 and ss <= 59 then
                time_parsed = string.format("%02d-%02d %02d:%02d:%02d", mm, dd, hh, mi, ss)
            end
        end
     end


     local v_sequenceId = buf(16,4)
     pkt.cols.protocol = "SGIP"
     local t = root:add(p_SGIP,buf(0,buf_len))
     t:add(f_Length,v_length)
     t:add(f_CommandId,v_command)
     t:add(f_NodeId,v_nodeId)
     t:add(f_Mmddhhmmss, v_mmddhhmmss):append_text(" (" .. time_parsed .. ")")
     t:add(f_SequenceId,v_sequenceId)
     --local dt = t:add(f_Data,buf(20,buf_len-20))
     v_command = v_command:uint()
     if v_command == 1 then
      SGIP_Bind(buf,pkt,t)
     elseif v_command == 3 then
      SGIP_Submit(buf,pkt,t)
     elseif v_command == 4 then
      SGIP_Deliver(buf,pkt,t)
     elseif v_command == 5 then
      SGIP_Report(buf,pkt,t)
     elseif v_command == 0x80000001 then
      SGIP_Bind_Response(buf,pkt,t)
     elseif v_command == 0x80000003 then
      SGIP_SubmitResponse(buf,pkt,t)
     elseif v_command == 0x80000004 then
      SGIP_DeliverResponse(buf,pkt,t)
     elseif v_command == 0x80000005 then
      SGIP_ReportResponse(buf,pkt,t)
     elseif v_command > 0x80000000 then
      SGIP_Response(buf,pkt,t)
     else
      t:add(f_Data,buf(20,buf_len-20))
     end
     return true
    end
    function p_SGIP.dissector(buf,pkt,root)
     -- 解析 SGIP
     if SGIP_dissector(buf,pkt,root) then
     else
      -- 使用默认输出
      data_dis:call(buf,pkt,root)
     end
    end

    tcp_table = DissectorTable.get("tcp.port")
    tcp_table:add(8801,p_SGIP)
   end
```
