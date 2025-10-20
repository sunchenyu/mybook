# smgp.lua脚本

```lua
-- SMGP.lua
-- SMGP protocol
-- author: h_Davy
do
	local p_SMGP = Proto("SMGP","SMGP","SMGP Protocol")
	local f_Length = ProtoField.uint32("SMGP.length","Packet Length",base.DEC)
	local command_id_enum = {
	   [1]="Login",      [0x80000001]="LoginResp",
	   [2]="Submit",     [0x80000002]="SubmitResp",
	   [3]="Deliver",    [0x80000003]="DeliverResp",
	   [4]="ActiveTest", [0x80000004]="ActiveTestResp",
	   [5]="Forward",    [0x80000005]="ForwardResp",
	   [6]="Exit",       [0x80000006]="ExitResp",
	   [7]="Query",      [0x80000007]="QueryResp"}
	local f_CommandId = ProtoField.uint32("SMGP.RequestId","Request ID",base.HEX,command_id_enum)
	local f_SequenceId = ProtoField.uint32("SMGP.sequenceId","Sequence ID",base.DEC);
	local f_ClientID = ProtoField.string("SMGP.ClientID","ClientID")
	local f_Authenticator = ProtoField.bytes("SMGP.Authenticator","Authenticator")
	local f_LoginMode = ProtoField.uint8("SMGP.LoginMode","Login Mode",base.DEC)
	local f_TimeStamp = ProtoField.uint32("SMGP.TimeStamp","TimeStamp",base.DEC)
	local f_Version = ProtoField.uint8("SMGP.Version","Version",base.HEX)
	local f_Status = ProtoField.uint32("SMGP.Status","Status",base.DEC,{[0]="OK"})
	local f_MsgType = ProtoField.uint32("SMGP.MsgType","MsgType",base.DEC,{[0]="MO",[6]="MT",[7]="P2P"})
	local f_NeedReport = ProtoField.uint32("SMGP.NeedReport","NeedReport",base.DEC,{[0]="N",[1]="Y"})
	local f_Priority = ProtoField.uint32("SMGP.Priority","Priority",base.DEC)
	local f_ServiceID = ProtoField.string("SMGP.ServiceID","ServiceID")
	local f_FeeType = ProtoField.string("SMGP.FeeType","FeeType")
	local f_FeeCode = ProtoField.string("SMGP.FeeCode","FeeCode")
	local f_FixedFee = ProtoField.string("SMGP.FixedFee","FixedFee")
	local f_MsgFormat = ProtoField.uint32("SMGP.MsgFormat","MsgFormat",base.DEC,{
	 [0]="ASCII",[3]="Card",[4]="Binary",[8]="UCS2",[15]="GB18030",[246]="SIM"})
	local f_ValidTime = ProtoField.string("SMGP.ValidTime","ValidTime")
	local f_AtTime = ProtoField.string("SMGP.AtTime","AtTime")
	local f_SrcTermID = ProtoField.string("SMGP.SrcTermID","SrcTermID")
	local f_ChargeTermID = ProtoField.string("SMGP.ChargeTermID","ChargeTermID")
	local f_DestTermIDCount = ProtoField.uint32("SMGP.DestTermIDCount","DestTermIDCount",base.DEC)
	local f_DestTermID = ProtoField.string("SMGP.DestTermID","DestTermID")
	local f_MsgLength = ProtoField.uint8("SMGP.MsgLength","MsgLength",base.DEC)
	local f_MsgContent = ProtoField.bytes("SMGP.MsgContent","MsgContent")
	local f_MsgID = ProtoField.bytes("SMGP.MsgID","MsgID")
	local f_IsReport = ProtoField.uint8("SMGP.IsReport","IsReport",base.DEC)
	local f_RecvTime = ProtoField.string("SMGP.RecvTime","RecvTime")
	local f_Reserve = ProtoField.string("SMGP.Reserve","Reserve")
	local f_TLV_Tag = ProtoField.uint16("SMGP.TLV_Tag", "TLV_Tag", base.HEX)
	local f_TLV_Length = ProtoField.uint16("SMGP.TLV_Length", "TLV Length", base.DEC)
	local f_TLV_Str_Value = ProtoField.string("SMGP.tlv_Value", "tlv Value")
	local f_TLV_Int_Value = ProtoField.uint16("SMGP.TLV_Value", "TLV Value",base.DEC)
    local f_DestSMGWNo = ProtoField.string("SMGP.DestSMGWNo","DestSMGWNo")
    local f_SrcSMGWNo = ProtoField.string("SMGP.SrcSMGWNo","SrcSMGWNo")
    local f_SMCNo = ProtoField.string("SMGP.SMCNo","SMCNo")
    local f_ReportFlag = ProtoField.bytes("SMGP.ReportFlag","ReportFlag")
	local p_TP_PID = Proto("TP_PID","TP_PID","TP_PID Protocol")
	local p_TP_UDHI = Proto("TP_UDHI","TP_UDHI","TP_UDHI Protocol")
	local p_Link_ID = Proto("Link_ID","Link_ID","Link_ID Protocol")
	local p_Pk_Total = Proto("Pk_Total","Pk_Total","Pk_Total Protocol")
	local p_Pk_Number = Proto("Pk_Number","Pk_Number","Pk_Number Protocol")

	
	p_SMGP.fields = {f_Length,f_CommandId,f_SequenceId,f_ClientID,f_Authenticator,
	f_LoginMode,f_TimeStamp,f_Version,f_Status,f_MsgType,f_NeedReport,f_Priority,
	f_ServiceID,f_FeeType,f_FeeCode,f_FixedFee,f_MsgFormat,f_ValidTime,f_AtTime,
	f_SrcTermID,f_ChargeTermID,f_DestTermIDCount,f_DestTermID,f_MsgLength,f_MsgContent,
	f_MsgID,f_IsReport,f_RecvTime,f_Reserve,f_TLV_Tag,f_TLV_Length,f_TLV_Str_Value,f_TLV_Int_Value,f_DestSMGWNo,f_SrcSMGWNo,f_SMCNo,f_ReportFlag}
	-- p_TP_PID.fields = {f_TLV_Tag,f_TLV_Length,f_TLV_Int_Value}
	-- p_TP_UDHI.fields = {f_TLV_Tag,f_TLV_Length,f_TLV_Int_Value}
	-- p_Link_ID.fields = {f_TLV_Tag,f_TLV_Length,f_TLV_Str_Value}
	-- p_Pk_Total.fields = {f_TLV_Tag,f_TLV_Length,f_TLV_Int_Value}
	-- p_Pk_Number.fields = {f_TLV_Tag,f_TLV_Length,f_TLV_Int_Value}
	
	local data_dis = Dissector.get("data")
	--
	local function SMGP_Login(buf,pkt,t)
	 t:add(f_ClientID,buf(12,8))
	 t:add(f_Authenticator,buf(20,16))
	 t:add(f_LoginMode,buf(36,1))
	 t:add(f_TimeStamp,buf(37,4))
	 t:add(f_Version,buf(41,1))
     pkt.cols.info:prepend("Login;" .. " ")
	end
	--
	local function SMGP_LoginResp(buf,pkt,t)
	 t:add(f_Status,buf(12,4))
	 t:add(f_Authenticator,buf(16,16))
	 t:add(f_Version,buf(32,1))
     pkt.cols.info:prepend("LoginResp;" .. " ")
	end
	--
	local function SMGP_Forward(buf,pkt,t)
		t:add(f_MsgID,buf(12,10))
		t:add(f_DestSMGWNo,buf(22,6))
		t:add(f_SrcSMGWNo,buf(28,6))
		t:add(f_SMCNo,buf(34,6))
		t:add(f_MsgType,buf(40,1))
		t:add(f_ReportFlag,buf(41,1))
		t:add(f_Priority,buf(42,1))
		t:add(f_ServiceID,buf(43,10))
		t:add(f_FeeType,buf(53,2))
		t:add(f_FeeCode,buf(55,6))
		t:add(f_FixedFee,buf(61,6))
		t:add(f_MsgFormat,buf(67,1))
		t:add(f_ValidTime,buf(68,17))
		t:add(f_AtTime,buf(85,17))
		t:add(f_SrcTermID,buf(102,21))
		t:add(f_DestTermID,buf(123,21))
		t:add(f_ChargeTermID,buf(144,21))
		pkt.cols.info:prepend("Forward;" .. " ")
		local v_msgLen = buf(165,1)
		t:add(f_MsgLength,v_msgLen)
		v_msgLen = v_msgLen:uint()
		t:add(f_MsgContent,buf(166,v_msgLen))
		--添加调用保留字段
		t:add(f_Reserve,buf(166 + v_msgLen,8))
	 	local tlv_offset = 174 + v_msgLen
		 -- 读取 TLV 参数
	  	while tlv_offset < buf:len() do

			local tlv_type = buf(tlv_offset, 2):uint()
			local tlv_length = buf(tlv_offset + 2, 2):uint()
			local tlv_value = buf(tlv_offset + 4, tlv_length)

			-- t:add(f_TLV_Tag, buf(tlv_offset, 2))
			-- t:add(f_TLV_Length, buf(tlv_offset + 2, 2))

			-- 根据 TLV 类型解析 Value
			if tlv_type == 0x0001 then

				local tlv_t = t:add(p_TP_PID, buf(tlv_offset, 4 + tlv_length))
				tlv_t:add(f_TLV_Tag, buf(tlv_offset, 2))
				tlv_t:add(f_TLV_Length, buf(tlv_offset + 2, 2))
				-- 假设类型 0x0001 是一个字符串
				tlv_t:add(f_TLV_Int_Value, buf(tlv_offset + 4, tlv_length))
			elseif tlv_type == 0x0002 then
				local tlv_t = t:add(p_TP_UDHI, buf(tlv_offset, 4 + tlv_length))
				tlv_t:add(f_TLV_Tag, buf(tlv_offset, 2))
				tlv_t:add(f_TLV_Length, buf(tlv_offset + 2, 2))
				-- 假设类型 0x0002 是一个整数
				tlv_t:add(f_TLV_Int_Value, tlv_value)
			elseif tlv_type == 0x000A then
				local tlv_t = t:add(p_Pk_Number, buf(tlv_offset, 4 + tlv_length))
				tlv_t:add(f_TLV_Tag, buf(tlv_offset, 2))
				tlv_t:add(f_TLV_Length, buf(tlv_offset + 2, 2))
				-- 假设类型 0x0002 是一个整数
				tlv_t:add(f_TLV_Int_Value, tlv_value)
			elseif tlv_type == 0x0009 then
				local tlv_t = t:add(p_Pk_Total, buf(tlv_offset, 4 + tlv_length))
				tlv_t:add(f_TLV_Tag, buf(tlv_offset, 2))
				tlv_t:add(f_TLV_Length, buf(tlv_offset + 2, 2))
				-- 假设类型 0x0002 是一个整数
				tlv_t:add(f_TLV_Int_Value, tlv_value)
			end

			-- 移动到下一个 TLV 参数
			tlv_offset = tlv_offset + 4 + tlv_length
		end
	end
	  --
	  local function SMGP_ForwardResp(buf,pkt,t)
		t:add(f_MsgID,buf(12,10))
		t:add(f_Status,buf(22,4))
		pkt.cols.info:prepend("ForwardResp;" .. " ")
	  end
	--
	local function SMGP_Submit(buf,pkt,t)
	 t:add(f_MsgType,buf(12,1))
	 t:add(f_NeedReport,buf(13,1))
	 t:add(f_Priority,buf(14,1))
	 t:add(f_ServiceID,buf(15,10))
	 t:add(f_FeeType,buf(25,2))
	 t:add(f_FeeCode,buf(27,6))
	 t:add(f_FixedFee,buf(33,6))
	 t:add(f_MsgFormat,buf(39,1))
	 t:add(f_ValidTime,buf(40,17))
	 t:add(f_AtTime,buf(57,17))
	 t:add(f_SrcTermID,buf(74,21))
	 t:add(f_ChargeTermID,buf(95,21))
	 t:add(f_DestTermIDCount,buf(116,1))
	 t:add(f_DestTermID,buf(117,21))
     pkt.cols.info:prepend("Submit;" .. " ")
	 local v_msgLen = buf(138,1)
	 t:add(f_MsgLength,v_msgLen)
	 v_msgLen = v_msgLen:uint()
	 t:add(f_MsgContent,buf(139,v_msgLen))
	 t:add(f_Reserve,buf(139 + v_msgLen,8))
	 local tlv_offset = 147 + v_msgLen
	 -- 读取 TLV 参数
	  while tlv_offset < buf:len() do

        local tlv_type = buf(tlv_offset, 2):uint()
        local tlv_length = buf(tlv_offset + 2, 2):uint()
        local tlv_value = buf(tlv_offset + 4, tlv_length)

        -- t:add(f_TLV_Tag, buf(tlv_offset, 2))
        -- t:add(f_TLV_Length, buf(tlv_offset + 2, 2))

        -- 根据 TLV 类型解析 Value
        if tlv_type == 0x0001 then

			local tlv_t = t:add(p_TP_PID, buf(tlv_offset, 4 + tlv_length))
			tlv_t:add(f_TLV_Tag, buf(tlv_offset, 2))
			tlv_t:add(f_TLV_Length, buf(tlv_offset + 2, 2))
            -- 假设类型 0x0001 是一个字符串
            tlv_t:add(f_TLV_Int_Value, buf(tlv_offset + 4, tlv_length))
        elseif tlv_type == 0x0002 then
			local tlv_t = t:add(p_TP_UDHI, buf(tlv_offset, 4 + tlv_length))
			tlv_t:add(f_TLV_Tag, buf(tlv_offset, 2))
			tlv_t:add(f_TLV_Length, buf(tlv_offset + 2, 2))
            -- 假设类型 0x0002 是一个整数
            tlv_t:add(f_TLV_Int_Value, tlv_value)
		elseif tlv_type == 0x000A then
			local tlv_t = t:add(p_Pk_Number, buf(tlv_offset, 4 + tlv_length))
			tlv_t:add(f_TLV_Tag, buf(tlv_offset, 2))
			tlv_t:add(f_TLV_Length, buf(tlv_offset + 2, 2))
            -- 假设类型 0x0002 是一个整数
            tlv_t:add(f_TLV_Int_Value, tlv_value)
		elseif tlv_type == 0x0009 then
			local tlv_t = t:add(p_Pk_Total, buf(tlv_offset, 4 + tlv_length))
			tlv_t:add(f_TLV_Tag, buf(tlv_offset, 2))
			tlv_t:add(f_TLV_Length, buf(tlv_offset + 2, 2))
            -- 假设类型 0x0002 是一个整数
            tlv_t:add(f_TLV_Int_Value, tlv_value)
        end

        -- 移动到下一个 TLV 参数
        tlv_offset = tlv_offset + 4 + tlv_length
	  end
    end
	--
	local function SMGP_SubmitResp(buf,pkt,t)
	 t:add(f_MsgID,buf(12,10))
	 t:add(f_Status,buf(22,4))
     pkt.cols.info:prepend("SubmitResp;" .. " ")
	end
	--
	local function SMGP_DeliverResp(buf,pkt,t)
		t:add(f_MsgID,buf(12,10))
		t:add(f_Status,buf(22,4))
		pkt.cols.info:prepend("DeliverResp;" .. " ")
	end
	--
	local function SMGP_Deliver(buf,pkt,t)
	 t:add(f_MsgID,buf(12,10))
	 local v_IsReport = buf(22,1)
	 t:add(f_IsReport,v_IsReport)
	 v_IsReport = v_IsReport:uint()
	 t:add(f_MsgFormat,buf(23,1))
	 t:add(f_RecvTime,buf(24,14))
	 t:add(f_SrcTermID,buf(38,21))
	 t:add(f_DestTermID,buf(59,21))
	 local v_msgLen = buf(80,1)
     pkt.cols.info:prepend("Deliver;" .. " ")
	 t:add(f_MsgLength,v_msgLen)
	 v_msgLen = v_msgLen:uint()
	 if v_IsReport == 1 then
	  --
	  t:add(f_MsgID,buf(84, 10)):append_text(' (Submit MsgID)')
	  t:add(f_MsgContent,buf(94,v_msgLen-13))
	 else
	  t:add(f_MsgContent,buf(81,v_msgLen))
	 end
	end
	--
	local function SMGP_ActiveTest( buf,pkt,root )
		pkt.cols.info:prepend("ActiveTest;" .. " ")
	end
	--
	local function SMGP_ActiveTestResp( buf,pkt,root )
		pkt.cols.info:prepend("ActiveTestResp;" .. " ")
	end
	--
	local function SMGP_dissector(buf,pkt,root)
	 local buf_len = buf:len();
	 if buf_len < 8 then return false end
	 local v_length = buf(0,4)
	--  v_length = v_length:uint()
	 local v_command = buf(4,4)
	 local v_sequenceId = buf(8,4)
	 pkt.cols.protocol = "SMGP"
	 local t = root:add(p_SMGP,v_length)
	 t:add(f_Length,v_length)
	 t:add(f_CommandId,v_command)
	 t:add(f_SequenceId,v_sequenceId)

	 --
	 v_command = v_command:uint()
	 if v_command == 1 then
	  SMGP_Login(buf,pkt,t)
	 elseif v_command == 2 then
	  SMGP_Submit(buf,pkt,t)
	 elseif v_command == 3 then
	  SMGP_Deliver(buf,pkt,t)
	 elseif v_command == 4 then
	  SMGP_ActiveTest(buf,pkt,t)
	 elseif v_command == 5 then
	  SMGP_Forward(buf,pkt,t)
	 elseif v_command == 0x80000001 then
	  SMGP_LoginResp(buf,pkt,t)
	 elseif v_command == 0x80000002 then
	  SMGP_SubmitResp(buf,pkt,t)
	 elseif v_command == 0x80000003 then
	  SMGP_DeliverResp(buf,pkt,t)
	 elseif v_command == 0x80000004 then
	  SMGP_ActiveTestResp(buf,pkt,t)
	 elseif v_command == 0x80000005 then
	  SMGP_ForwardResp(buf,pkt,t)
	 elseif v_command > 0x80000000 then
	  --
	 else
	  t:add(f_Data,buf(20,buf_len-20))
	 end
	 return true
	end
	--
	function p_SMGP.dissector(buf,pkt,root)
	 -- if SMGP_dissector(buf,pkt,root) then
     -- else
     --  data_dis:call(buf,pkt,root)
     --将下面那段替换上面的那段
     --从这里开始
     local buf_len = buf:len()
	 local len_start = 0
	 repeat
        if len_start >= buf_len then
            break
        end

        local len_end = buf(len_start, 4)
        if len_end:len() < 4 then
            break
        end
        len_end = len_end:uint()

        if len_end > buf_len - len_start then
            break
        end

        SMGP_dissector(buf(len_start, len_end), pkt, root)

        len_start = len_start + len_end
        shengyu_length = buf_len - len_start

    until shengyu_length <= 8

    if shengyu_length > 0 then
        data_dis:call(buf(len_start, shengyu_length), pkt, root)
    end
	--  if SMGP_dissector(buf,pkt,root) then
	--  else
	--   data_dis:call(buf,pkt,root)
	--  end
	end
	
	tcp_table = DissectorTable.get("tcp.port")
	tcp_table:add(8890,p_SMGP)
	tcp_table:add(8900,p_SMGP)
   endome code
```
