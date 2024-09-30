# 解析复杂日志中的字段

## 场景一：从复杂字符串中解析出json字符串，并展开特定字段
  * 原始日志
    ```
    content: [v1] [2024-07-12 14:45:15.497+0800] [INFO ] [XXID-1 task-10] WBERVIE [UID: N/A] m_point|{\"extra\":\"{\\\"items\\\":[{\\\"path\\\":\\\"Vdsjbxk.Cbsj.EV.FPDD\\\",\\\"id\\\":17,\\\"value\\\":1}]}\",\"vin\":\"WQ21497492\",\"source\":\"clo_is\",\"event\":\"clo_edd_received_rop\",\"tid\":\"Rcslcml\",\"ts\":1720680315491}
    ```
  * 解析需求
      * 1：从content字段中提取出m_point后面的json对象，并拆为键值对。
      * 2：通过判断event的值给新字段newid、pid、pid_type、cid、cid_type设置值。
      * 3：再把从1中解析出来的时间字段ts的值设为当前的__time__
  * 加工语句
    ```python
    * | project content 
    | where regexp_like(content, '.*m_point\|.*') 
    | parse-regexp content, '.*m_point\|(.*)' as a 
    | parse-json a 
    | project-away content , a 
    | extend newid = if(event like 'clo_edd_received_rop', 'T989092', '') ,  pid = if(event like 'clo_edd_received_rop', 'clo_edd_received_rop', '') ,  pid_type = if(event like 'clo_edd_received_rop', 'clo_edd_received_rop', '') ,  cid = if(event like 'clo_edd_received_rop', 'clo_edd_received_rop', '') , cid_type = if(event like 'clo_edd_received_rop', 'clo_edd_received_rop', '') 
    |extend "__time__" = cast(ts as bigint)/1000
    ```
  * 对应结果
    ```
    cid:clo_edd_received_rop
    cid_type:clo_edd_received_rop
    event:clo_edd_received_rop
    extra:{"items":[{"path":"Vdsjbxk.Cbsj.EV.FPDD","id":17,"value":1}]}
    newid:T989092
    pid:clo_edd_received_rop
    pid_type:clo_edd_received_rop
    source:clo_is
    tid:Rcslcml
    ts:1720680315491
    vin:WQ21497492
    ```
## 场景二：从含有xml字符的字符串中解析出json对象，并二级展开。
  * 原始日志
    ```
    {"__log_order_key__":11,"traceId":"dnohdohwhwiqnd923010hem2e","duration":"342424","spanId":"cr334c44","parentSpanId":"0","startTime":"1713765089425444896","spanName":"/DSFEF/RulePort","refefd":"csai-uat","pid":"kru2w@ere","ip":"","kind":"1","hostname":"","statusCode":"0","statusMessage":"","traceState":"","attributes":"{\"serviceType\":\"unknown\",\"db.erere\":\"unknown\",\"pid\":\"e33feeeeeeeeef234423\",\"source\":\"ebpf\",\"clusterId\":\"h8fhih9h9h99eh8hief\",\"resp.header\":\"Content-Type: text/xml;charset=utf-8\\nDate: Mon, 22 Apr 2024 05:51:29 GMT\\nServer: Apache-Coyote/1.1\\n\",\"status.code\":\"200\",\"container.id\":\"93d02646a21289224e210abd12c3988660c2dfeea3a2151742014249a5d55844\",\"callType\":\"http_client\",\"source_ip\":\"172.22.7.179\",\"resp.body\":\"<?xml version=\\\"1.0\\\" ?><S:Tvelop xmlns:S=\\\"http://schemas.xml.org/soap/Tvelope/\\\"><S:Body><ns2:ddResponse xmlns:ns2=\\\"http://service.tt.sinosoft.com/\\\"><return>{&quot;applicationNum&quot;:&quot;232323232&quot;,&quot;businessModule&quot;:&quot;UW&quot;,&quot;isApproved&quot;:&quot;0&quot;,&quot;isTest&quot;:false,&quot;subSystem&quot;:&quot;CS&quot;,&quot;system&quot;:&quot;MICO&quot;,&quot;verifyResultList&quot;:[{&quot;flag&quot;:&quot;2&quot;,&quot;returnInfo&quot;:&quot;请确认是否继续。&quot;,&quot;ruleCategories&quot;:&quot;2&quot;,&quot;ruleName&quot;:&quot;CSUW0300001-双录质检结果校验&quot;}]}</return></ns2:ddResponse></S:Body></S:Tvelop>\",\"endpoint\":\"/CSRuleintface/RulePort\",\"cmonitor\":\"KSpanInfo\",\"addr_family\":\"0\",\"remote_ip\":\"10.9.1.23\",\"req.body\":\"<?xml version='1.0' encoding='UTF-8'?><S:Tvelop xmlns:S=\\\"http://schemas.xml.org/soap/Tvelop/\\\"><S:Body><ns2:fireRule xmlns:ns2=\\\"http://service.t't.sinosoft.com/\\\"><arg0>MICO</arg0><arg1>CS</arg1><arg2>UW</arg2><arg3>{\\\"csuw\\\":{\\\"applicationNum\\\":\\\"232323232\\\",\\\"policyID\\\":\\\"213u92heijodwdwq3e231\\\",\\\"policyNum\\\":\\\"0839820984004\\\",\\\"CSType\\\":\\\"108\\\",\\\"applicationDate\\\":\\\"2024-04-22\\\",\\\"systemDate\\\":\\\"2024-04-22\\\",\\\"submitChannel\\\":\\\"12\\\",\\\"CheckIdent\\\":\\\"2\\\",\\\"CSAcceptanceNum\\\":\\\"00002030087801029\\\",\\\"organCode\\\":\\\"80040204\\\",\\\"applicantList\\\":[{\\\"holderID\\\":\\\"\\\",\\\"applicantName\\\":\\\"胡凯强\\\",\\\"applyAge\\\":26,\\\"sex\\\":\\\"M\\\",\\\"birthDate\\\":\\\"1997-10-20\\\",\\\"IDCardType\\\":\\\"1\\\",\\\"cardID\\\":\\\"110106199710200614\\\",\\\"nationality\\\":\\\"37\\\",\\\"occupationCode\\\":\\\"J001001\\\",\\\"address\\\":\\\"\\\",\\\"telephone\\\":\\\"\\\",\\\"hasInform\\\":\\\"N\\\",\\\"customerInformList\\\":[{\\\"ordinalNum\\\":\\\"13\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"15.2\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.2\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.9\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"9\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.8\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.6\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"8\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"7\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.4\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"2\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.12\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.7\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.1\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.13\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.11\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"15.1\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"4\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"5\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"3\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"6\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"14\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.5\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"12\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.3\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.10\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"1\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"15.3\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"11\\\",\\\"hasInform\\\":\\\"2\\\"}]}],\\\"insurantList\\\":[{\\\"insurantID\\\":\\\"f65239503f52407e9806efb5d3b054f4\\\",\\\"insurantName\\\":\\\"行\\\",\\\"applyAge\\\":25,\\\"sex\\\":\\\"F\\\",\\\"birthDate\\\":\\\"1998-11-05\\\",\\\"IDCardType\\\":\\\"3\\\",\\\"cardID\\\":\\\"246558761420\\\",\\\"nationality\\\":\\\"1\\\",\\\"occupationCode\\\":\\\"J001001\\\",\\\"address\\\":\\\"\\\",\\\"telephone\\\":\\\"\\\",\\\"hasInform\\\":\\\"N\\\",\\\"weight\\\":50.0,\\\"height\\\":165.0,\\\"healthInformList\\\":[{\\\"hasSmoking\\\":\\\"N\\\",\\\"dailySmokAmount\\\":0.0,\\\"smokeYear\\\":0.0,\\\"hasDrinking\\\":\\\"N\\\",\\\"drinkType\\\":\\\"酒\\\",\\\"dailyDrinkAmount\\\":0.0,\\\"drinkYear\\\":0.0}],\\\"productList\\\":[{\\\"productCode\\\":\\\"10131011\\\",\\\"productID\\\":\\\"\\\",\\\"productName\\\":\\\"附加保险\\\",\\\"isMainRisk\\\":\\\"0\\\",\\\"amount\\\":5000.0,\\\"copies\\\":0},{\\\"productCode\\\":\\\"10132005\\\",\\\"productID\\\":\\\"\\\",\\\"productName\\\":\\\"附加医疗\\\",\\\"isMainRisk\\\":\\\"0\\\",\\\"amount\\\":1800.0,\\\"copies\\\":1}],\\\"beneficiaryList\\\":[{\\\"benefitID\\\":\\\"\\\",\\\"beneficiaryName\\\":\\\"\\\",\\\"applyAge\\\":0,\\\"sex\\\":\\\"\\\",\\\"birthDate\\\":\\\"9999-12-31\\\",\\\"IDCardType\\\":\\\"\\\",\\\"cardID\\\":\\\"\\\",\\\"nationality\\\":\\\"\\\",\\\"occupationCode\\\":\\\"\\\",\\\"address\\\":\\\"\\\",\\\"telephone\\\":\\\"\\\"}],\\\"customerInformList\\\":[{\\\"ordinalNum\\\":\\\"13\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"15.2\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.2\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.9\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"9\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.8\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.6\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"8\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"7\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.4\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"2\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.12\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.7\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.1\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.13\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.11\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"15.1\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"4\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"5\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"3\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"6\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"14\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.5\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"12\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.3\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"10.10\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"1\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"15.3\\\",\\\"hasInform\\\":\\\"2\\\"},{\\\"ordinalNum\\\":\\\"11\\\",\\\"hasInform\\\":\\\"2\\\"}]}],\\\"agentList\\\":[{\\\"agentCode\\\":\\\"100606285\\\",\\\"agentName\\\":\\\"肥大于一\\\",\\\"agentAge\\\":65,\\\"agentSex\\\":\\\"M\\\",\\\"agentBirth\\\":\\\"1959-01-01\\\",\\\"IDCardType\\\":\\\"1\\\",\\\"cardID\\\":\\\"370983195901019996\\\",\\\"agentChannel\\\":\\\"\\\",\\\"telephone\\\":\\\"\\\"}]}}</arg3><arg4>false</arg4></ns2:fireRule></S:Body></S:Tvelop>\",\"source_port\":\"49152\",\"cmonitor_name\":\"cmonitor-agent-9xmdd\",\"host\":\"172.22.7.179\",\"cmonitor_ip\":\"10.3.72.9\",\"net_ns\":\"32232223\",\"ali.trace.rpc\":\"/CSRuleintface/RulePort\",\"deployment\":\"cd-insurance\",\"app\":\"csai--uat\",\"k8s.pod.ip\":\"172.22.7.179\",\"workloadKind\":\"deployment\",\"component.name\":\"http\",\"rpc\":\"/CSRuleintface/RulePort\",\"remote_port\":\"8010\",\"workloadName\":\"cd-insurance\",\"process_pid\":\"364469\",\"tsid\":\"25887099391057281\",\"version\":\"HTTP/1.1\",\"pod_name\":\"cd-insurance-787b759797-lchbk\",\"http.status_code\":\"200\",\"req.header\":\"Content-Type: text/xml; charset=utf-8\\nSoapaction: \\\"\\\"\\nUser-Agent: JAX-WS RI 2.2.9-b130926.1035 svn-revision#jd2939hondohd9y9hd2313\\nConnection: keep-alive\\nContent-Length: 4212\\nAccept: text/xml, multipart/related\\n\",\"rootIp\":\"172.22.7.179\",\"peer_k8s.pod.ip\":\"10.9.1.23\",\"destId\":\"10.9.1.23\",\"slow\":\"1\",\"service\":\"csai-29jodejnkn-insurance-uat\",\"ali.trace.flag\":\"x-trace\",\"namespace\":\"csai-ts\",\"ali.trace.wmjjelwl\":\"98\",\"serverIp\":\"172.22.7.179\",\"fd\":\"45\",\"statusCode\":\"200\",\"rpcType\":\"25\"}","resources":"{\"service.name\":\"cd-insurance\"}","__pack_meta__":"1|jiewojoj==|29|7","__topic__":"","__source__":"172.17.99.3","__tag__:__pack_id__":"9034804828-53FA94","__time__":"1713765092"}
    ```
  * 解析需求
      * 1：从attributes字段中提取出req.body的值。
      * 2：再从req.body中取出xml字符串中包裹的json字符串
      * 3：再把从2中解析出来的json字符串二级展开
  * 加工语句
    ```python
    * | project attributes 
    | extend a = json_extract(attributes, '$["req.body"]') 
    | extend b=regexp_extract(try_cast(a as varchar), '<arg3>(.*)<\/arg3>',1) 
    | project b|parse-json -path='$.csuw' b 
    | project-away b
    ```
  * 对应结果
    ```
    CSAcceptanceNum:00002030087801029
    CSType:108
    CheckIdent:2
    agentList:[{"telephone":"","agentChannel":"","cardID":"370983195901019996","IDCardType":"1","agentBirth":"1959-01-01","agentSex":"M","agentAge":65,"agentName":"肥大于一","agentCode":"100606285"}]
    applicantList:[{"customerInformList":[{"hasInform":"2","ordinalNum":"13"},{"hasInform":"2","ordinalNum":"15.2"},{"hasInform":"2","ordinalNum":"10.2"},{"hasInform":"2","ordinalNum":"10.9"},{"hasInform":"2","ordinalNum":"9"},{"hasInform":"2","ordinalNum":"10.8"},{"hasInform":"2","ordinalNum":"10.6"},{"hasInform":"2","ordinalNum":"8"},{"hasInform":"2","ordinalNum":"7"},{"hasInform":"2","ordinalNum":"10.4"},{"hasInform":"2","ordinalNum":"2"},{"hasInform":"2","ordinalNum":"10.12"},{"hasInform":"2","ordinalNum":"10.7"},{"hasInform":"2","ordinalNum":"10.1"},{"hasInform":"2","ordinalNum":"10.13"},{"hasInform":"2","ordinalNum":"10.11"},{"hasInform":"2","ordinalNum":"15.1"},{"hasInform":"2","ordinalNum":"4"},{"hasInform":"2","ordinalNum":"5"},{"hasInform":"2","ordinalNum":"3"},{"hasInform":"2","ordinalNum":"6"},{"hasInform":"2","ordinalNum":"14"},{"hasInform":"2","ordinalNum":"10.5"},{"hasInform":"2","ordinalNum":"12"},{"hasInform":"2","ordinalNum":"10.3"},{"hasInform":"2","ordinalNum":"10.10"},{"hasInform":"2","ordinalNum":"1"},{"hasInform":"2","ordinalNum":"15.3"},{"hasInform":"2","ordinalNum":"11"}],"hasInform":"N","telephone":"","address":"","occupationCode":"J001001","nationality":"37","cardID":"110106199710200614","IDCardType":"1","birthDate":"1997-10-20","sex":"M","applyAge":26,"applicantName":"胡凯强","holderID":""}]收起
    applicationDate:2024-04-22
    applicationNum:232323232
    insurantList:[{"beneficiaryList":[{"telephone":"","address":"","occupationCode":"","nationality":"","cardID":"","IDCardType":"","birthDate":"9999-12-31","sex":"","applyAge":0,"beneficiaryName":"","benefitID":""}],"productList":[{"copies":0,"amount":5000,"isMainRisk":"0","productName":"附加保险","productID":"","productCode":"10131011"},{"copies":1,"amount":1800,"isMainRisk":"0","productName":"附加医疗","productID":"","productCode":"10132005"}],"healthInformList":[{"drinkYear":0,"dailyDrinkAmount":0,"drinkType":"酒","hasDrinking":"N","smokeYear":0,"dailySmokAmount":0,"hasSmoking":"N"}],"height":165,"hasInform":"N","telephone":"","address":"","occupationCode":"J001001","nationality":"1","IDCardType":"3","birthDate":"1998-11-05","insurantID":"f65239503f52407e9806efb5d3b054f4","customerInformList":[{"hasInform":"2","ordinalNum":"13"},{"hasInform":"2","ordinalNum":"15.2"},{"hasInform":"2","ordinalNum":"10.2"},{"hasInform":"2","ordinalNum":"10.9"},{"hasInform":"2","ordinalNum":"9"},{"hasInform":"2","ordinalNum":"10.8"},{"hasInform":"2","ordinalNum":"10.6"},{"hasInform":"2","ordinalNum":"8"},{"hasInform":"2","ordinalNum":"7"},{"hasInform":"2","ordinalNum":"10.4"},{"hasInform":"2","ordinalNum":"2"},{"hasInform":"2","ordinalNum":"10.12"},{"hasInform":"2","ordinalNum":"10.7"},{"hasInform":"2","ordinalNum":"10.1"},{"hasInform":"2","ordinalNum":"10.13"},{"hasInform":"2","ordinalNum":"10.11"},{"hasInform":"2","ordinalNum":"15.1"},{"hasInform":"2","ordinalNum":"4"},{"hasInform":"2","ordinalNum":"5"},{"hasInform":"2","ordinalNum":"3"},{"hasInform":"2","ordinalNum":"6"},{"hasInform":"2","ordinalNum":"14"},{"hasInform":"2","ordinalNum":"10.5"},{"hasInform":"2","ordinalNum":"12"},{"hasInform":"2","ordinalNum":"10.3"},{"hasInform":"2","ordinalNum":"10.10"},{"hasInform":"2","ordinalNum":"1"},{"hasInform":"2","ordinalNum":"15.3"},{"hasInform":"2","ordinalNum":"11"}],"weight":50,"cardID":"246558761420","sex":"F","applyAge":25,"insurantName":"行"}]收起
    organCode:80040204
    policyID:213u92heijodwdwq3e231
    policyNum:0839820984004
    submitChannel:12
    systemDate:2024-04-22
    ```