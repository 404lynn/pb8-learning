# pb8-learning
powerbuilder8的知识库，记录一下学习的东西
# 一些函数 
1.网络连接的
    string ls_data

    String ls_url                   ='http://10.141.59.222:9857/cc_sendtowx'
    OleObject lole   
    lole = CREATE oleobject 
    lole.ConnectToNewObject("Microsoft.XMLHttp") 
    lole.open("POST",ls_url, false)   
    lole.setRequestHeader("Content-type", "application/json; encoding=utf-8")
    lole.setRequestHeader("accept","*/*")
    lole.setRequestHeader("connection","Keep-Alive")
    lole.setRequestHeader("user-agent","Mozilla/4.0(compatible; MSIE 6.0; Windows NT 5.1;SV1)")
    lole.setRequestHeader('Content-Length',string(len(ls_data)))
    String ls_json
    ls_json = '{"isgroup":"' + isgroup + '","getname":"' + getname + '","url":"' + url + '","content":"' + content + '","atpeople":"' + atpeople + '"}'
    ls_data=ls_json
    if len(ls_data)>0 then
        lole.send(ls_data)
    end if
    destroy lole 
    需要换的时候改改url和传的东西
2.连接大模型api

    函数名称：f_deepseek_chat_completion
    功能描述：调用DeepSeek大模型API
    参数说明：
        as_api_key  - API密钥
        as_model     - 模型名称
        as_system_content - 系统角色内容 
        as_user_question - 用户问题
    返回值：string - 返回模型生成内容
    string ls_response, ls_data

    // API基础地址
    string ls_url = "https://api.deepseek.com/v1/chat/completions"

    // 创建Ole对象
    OleObject lole
    lole = CREATE OleObject
    lole.ConnectToNewObject("Microsoft.XMLHttp")

    // 配置请求
    lole.open("POST", ls_url, FALSE)

    // 设置请求头
    lole.setRequestHeader("Content-Type", "application/json")
    lole.setRequestHeader("Authorization", "Bearer " + as_api_key)
    lole.setRequestHeader("Accept", "application/json")

    // 构建请求体
    string ls_json
    ls_json = '{"model":"' + as_model + '",' + &
            '"messages":[' + &
            '{"role":"system","content":"' + as_system_content + '"},' + &
            '{"role":"user","content":"' + as_user_question + '"}' + &
            ']}'

    ls_data = ls_json

    // 发送请求
    lole.send(ls_data)

    // 获取响应
    ls_response = lole.responseText

    // 解析响应内容（简单字符串处理-这可以用json。）
    string ls_content,ls_char
    integer li_pos_start, li_pos_end,li_pos_end2,li_i,li_pos_end3
        li_pos_start = Pos(ls_response, '"content":"', 1)
        li_pos_end = Pos(ls_response, "finish_reason", 1) 
        li_pos_end2 = Pos(ls_response, "logprobs", 1) 
        li_pos_end3 = Pos(ls_response, "reasoning_content", 1) 
        if li_pos_end3 < li_pos_end2  and li_pos_end3 > li_pos_start then
            li_pos_end2 = li_pos_end3
        end if
        if li_pos_end2 < li_pos_end  and li_pos_end2 > li_pos_start then
            li_pos_end = li_pos_end2 
        end if
        for li_i = li_pos_start + 11 to li_pos_end  -5
            ls_char = Mid(ls_response, li_i, 1)
            ls_content = ls_content+ls_char
        next
    // 清理对象
    DESTROY lole

    RETURN ls_content
# 数据窗口的sql如何比较日期
    CONVERT(varchar, chattime, 23)>=:ls_kssj and CONVERT(varchar, chattime, 23)<=:ls_jssj 
    其中ls_jssj的值string(relativedate(today(),-30),"yyyy-mm-dd")
    比较年月日的方法
# 获取datetime在pb里面
    ldt_t1 = datetime(date(string(today(),"yyyy-mm-dd"))
    ,time("00:00:00"))
# 使用结构体传参
    if row < 1 then return
    string sql1
    stcc_hd stcc_hd
    string ls_hdbh,ls_qmc1
    datawindowchild l_child
    if dwo.name='xq' then
        if isnull(this.object.qm[row]) then
            messagebox("注意","请选择群聊!")
            stcc_hd.qmc1 = "没有选择群号"
            stcc_hd.hdbh = this.object.hdbh[row]
        else
            stcc_hd.qm = this.object.qm[row]
            stcc_hd.hdbh = this.object.hdbh[row]
            select qmc1 into :ls_qmc1 from hdtxz where qm = :stcc_hd.qm and hdbh =:stcc_hd.hdbh;
            stcc_hd.qmc1 = ls_qmc1
        end if
        if isnull(this.object.hdbh[row]) then
            messagebox("注意","请先保存!")
        else
            stcc_hd.hdkssj = this.object.hdkssj[row]
            stcc_hd.ysjd = this.object.ysjd[row]
            stcc_hd.hdjssj = this.object.hdjssj[row]
            openwithparm(wc_wxc,stcc_hd)
            
        end if

    end if
#	复制数据窗口数据
    wj_rcsc.reset()  情况
    dwj_rcsc2.RowsCopy(1, dwj_rcsc2.RowCount(), Primary!, dwj_rcsc, 1, Primary!) 复制过去
    long ll_row, ll_rows
    ll_rows = dwj_rcsc.RowCount()

    FOR ll_row = 1 to ll_rows
        dwj_rcsc.SetItemStatus(ll_row, 0, Primary!,DataModified!)  复制的不要跟新
    NEXT