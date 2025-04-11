- [pb8-learning](#pb8-learning)
- [一些函数](#一些函数)
- [数据窗口的sql如何比较日期](#数据窗口的sql如何比较日期)
- [获取datetime在pb里面](#获取datetime在pb里面)
- [使用结构体传参](#使用结构体传参)
- [复制数据窗口数据](#复制数据窗口数据)
- [datetime的东西](#datetime的东西)
  - [获取当前的datetime](#获取当前的datetime)
  - [string转换datetime](#string转换datetime)
  - [获取今天的datetime](#获取今天的datetime)
  - [datetime下日期加1](#datetime下日期加1)
- [数据窗口子窗口模糊搜索](#数据窗口子窗口模糊搜索)
- [前端的知识](#前端的知识)
  - [怎么在调试的时候跳过验证](#怎么在调试的时候跳过验证)
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
#   datetime的东西
## 获取当前的datetime
    now_time = datetime(date(string(today(),"yyyy-mm-dd")), now())
## string转换datetime
    ld_time = datetime(date(left(ls_datetime,10)),time(right
    (ls_datetime,len(ls_datetime) - 11)))
    其中ls_datetime这个要满足yyyy-mm-dd hh:mm:ss
## 获取今天的datetime
    datetime(date(left(ls_datetime,10)),time("00:00:00"))

## datetime下日期加1
    其中ld_today为datetime格式
    ld_today = dw_jxjjgl.object.qsj[ll_selectedRow]
    // 分解日期和时间
    date ld_date,ld_oldtime
    time ld_time
    datetime ld_new_date
    ld_date = Date(ld_today)  // 提取日期部分
    ld_time = Time(ld_today)  // 提取时间部分
    // 日期加一天
    ld_date = RelativeDate(ld_date, 1)  
    // 或 ld_date = ld_date + 1
    // 合并为新日期时间
    ld_new_date = DateTime(ld_date, ld_time)
# 数据窗口子窗口模糊搜索
    call super::editchanged;if row < 1 then return
    string ls_filter,ls_filter2,ls_filter3
    点击什么字段执行
    if dwo.name="cc_ygid" then
    设置子窗口
        datawindowchild dwc
        long ll_c
        谁的子窗口
        ll_c = this.getchild("ygid",dwc)
        if ll_c <> - 1 then
            if isnull(data) or len(data)=0 then
                dwc.setfilter("")//bh like 'HC%'
                dwc.filter()
            else//(bh like 'HC%') and
            ls_filter =  "(ygid like '%" + data + "%')"
                ls_filter2 =  "(ygxm like '%" + data + "%')"
                ls_filter3 = ls_filter2 + " or " + ls_filter
                这只针对该字段显示的子字段有效
    //			dwc.setfilter(" ((ygid like '%"+data+"%') or  (ygxm like '%"+data+"%'))")
                dwc.SetFilter(ls_filter3)
                dwc.filter()
            end if
            dwc.setsort("ygid a")
            dwc.sort()
        end if
    end if
# 前端的知识
## 怎么在调试的时候跳过验证
	const IS_DEV = true
	// const API_MP = IS_DEV ? 'http://10.141.59.53:8083/zwwlmp' : 'https://shouji.zwwl56.com/zwwlmp';
	const API_MP = 'http://10.141.59.53:8083/zwwlmp';
    这个IS_DEV要调成true api换换
    登录的照抄
    this.apiRequest(zwutil.zwrequest({
				url: `${API_MP}/erp/cyltj?dateFrom=${this.dateFrom}&dateTo=${this.dateTo}`,
				method: 'GET',
				//notoken: true,
			}).then(r => {
				if (r.code === 0) {
					this.getdata = r.data
					console.log('LOGIN:', this.getdata)
				}
				return r
			}))
    注意要有token的要加上
