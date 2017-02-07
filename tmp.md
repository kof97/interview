API Document -- GDT ADS
===================
文档版本：

	* 建议用Firefox，chrome及IE9以上查看此文档


#一、使用入门
************************************************
本文将介绍如何使用ADS API接入广点通强大的系统能力，实现更加高效灵活的广告投放。 ADS API覆盖了广点通系统的各个层级，包括  

- **基础能力部分**：广告主模块、账户模块，推广计划模块、广告组模块、素材模块；

- **定向能力部分**：定向模块、自定义人群模块  

- **报表能力部分**：报表模块、离线报表模块、混合视图报表模块

- **服务商部分**：服务商模块


其中直客能够调用除“服务商部分”以外的其它部分。 另外，获得API授权的服务商自动获得了其子客户账户的API操作权限。 


##1.1 通用说明 
************************************************
- 参数编码请使用UTF-8格式

- 接口所述时间戳如无特殊说明，精确到秒即可

- 为保证接口服务质量，系统会对接口单位时间请求次数做限制，根据业务的接口需求，配置相应的接口配额。业务接入方请联系 derekxwang 确定接口配额事宜。



##1.2 API基本结构
************************************************

	沙箱环境使用如下域名
	http://sandbox.api.e.qq.com

	#正式环境直接使用api.e.qq.com 访问,并使用https协议
	#沙箱环境下的数据操作账户是真实账户，但沙箱环境的数据与正式环境的数据相隔离

	接口路径：
		http://api.e.qq.com/ads/{version}/{mod}/{act}/
		version:版本号码,当前版本号码为 v3
		mod:模块名称add
		act:动作名称

		如，获取广告主信息,接口名称为 advertiser/read，地址为： http://api.e.qq.com/ads/v3/advertiser/read 

##1.3 请求方式
************************************************	
	由于部分客户端（如较老版本的浏览器）的功能限制，请求方式可简化为：
	GET 用来获取资源（默认方式，可省略）；
	POST 用来增、删、改资源；
	简化后的 POST 承担更新工作，

	请求内容则可以参考详细接口文档。

##1.4 响应
************************************************	
	所有的响应信息都包含在返回的Json中，包括本次操作的返回码（ret）以及错误信息（msg）。
	
	响应 JSON 中，必填的字段有：
	ret 返回码，非 0 则为错误码，可参考附录 错误码及其定义
	msg 错误信息，当 ret 非 0 时有值
	data 资源的数据根节点
	
##1.5 输出格式
************************************************		
	format:     返回数据的格式(可选)，当前仅支持json；
	charset:    返回数据的编码(可选)，GBK|UTF-8，默认UTF-8 
	unicode:    设置后 所有非ASCII字符都用\uxxxx表示，原生的JSON  （对于代码中接口调用建议加上此项） (可选)   
	post_format: 'json':以JSON格式传入数据, 'default':标准POST格式(默认值)      (可选)

##1.6 API认证机制
************************************************
	==========================服务商/直客==========================

	token: 授权令牌 (必须) token=base64(uid,appid,time,sign),所有内容使用半角逗号(,)按顺序拼接起来，再经过Base64编码
 
		其中：
			uid: 授权客户UID，仅支持代理商或直客广告主授权，通常与appid一致（授权申请请联系derekxwang）
			appid：安全凭证appid，即授权接入方唯一身份标识
			time：发起请求时的unix时间戳，这个值跟接收到请求时的服务器时间戳值偏差（正负）,
				  超过1200秒（20分钟）时，请求会被拒绝，要求调用方重新生成token（请使用Asia/Shanghai时区）。
			sign：签名字符串，sign = sha1(appid+appkey+time)，没有加号+，采用sha1加密算法生成签名串
				  其中appid和appkey为授权后获得的标识ID和私钥KEY,time为上面使用的unix时间戳

		举例说明（以PHP语言举例）：
		  \$uid = 12345;
		  \$appid = "12345";
		  \$appkey = "032742947398^*";
		  \$time = time();

		  step 1:生成sign
			\$sign = sha1(\$appid.\$appkey.\$time);
				 = sha1("12345032742947398^*1388478907")
				 = ec73cad58c1f8ade4b50a6e0b36d3835d7427b5c
		  step 2:生成token
			\$token = base64_encode(\$uid . ',' . \$appid . ',' . \$time . ',' . \$sign);
				  = base64_encode("12345,12345,1388478907,ec73cad58c1f8ade4b50a6e0b36d3835d7427b5c")
				  = MTIzNDUsMTIzNDUsMTM4ODQ3ODkwNyxlYzczY2FkNThjMWY4YWRlNGI1MGE2ZTBiMzZkMzgzNWQ3NDI3YjVj

		  这样，我们就生成了本次api请求的token了


	format:     返回数据的格式(可选)，当前仅支持json；
	charset:    返回数据的编码(可选)，GBK|UTF-8，默认UTF-8
	unicode:    设置后 所有非ASCII字符都用\uxxxx表示，原生的JSON  （对于代码中接口调用建议加上此项） (可选)  
	post_format: 'json':以JSON格式传入数据, 'default':标准POST格式(默认值)      (可选)
	 
	以上接口参数均以GET方式传递,输入数据的格式默认为UTF8。


##1.7 特别注意
************************************************
	1. 当广告投放站点非移动站点（SITESET_MOBILE_UNION,SITESET_MOBILE_INNER)，且使用移动相关定向时（操作系统，联网方式，运营商等），
	则将导致广告曝光异常
	2. 部分注明[京东专用]的接口仅供京东特定账号调用，其它调用方调用将导致异常。

#二、指南		

##2.1 认证授权模块

认证授权模块描述

###2.1.1 OAuth2.0使用authorization_code换取access_token
拉取账户信息
************************************************
	地址：auth/access_token
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>app_id</td>
			<td>string</td>
			<td>合作方APP ID</td>
			<td>小于32字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>app_key</td>
			<td>string</td>
			<td>密钥APP KEY</td>
			<td>小于32字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>authorization_code</td>
			<td>string</td>
			<td>authorization_code，用于换取access_token</td>
			<td>不大于64个英文字符</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/auth/access_token?app_id=50254&app_key=311fcd3d81757e28a5409bd403313461&authorization_code=3200d95fe0647a8a9cb628d02840c714" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>access_token</td>
			<td>string</td>
			<td>access_token用于访问受限资源</td>
		</tr>
		<tr>
			<td>expires_in</td>
			<td>integer</td>
			<td>有效期时间</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": '',
			"data": {
					"access_token":"3d029a272df143d782db524fc1cbddcb",	
					"expires_in":86400		//有效期
			}
	}
										
```
##2.2 广告主模块

###2.2.1 合并advertiser/read和agency/get_advertiser_list
************************************************
	地址：advertiser/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>no</td>
		</tr>
		<tr>
			<td>filter</td>
			<td>array</td>
			<td>过滤条件</td>
			<td>若此字段不传，或传空则视为无限制条件</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/advertiser/get?access_token=<TOKEN>&page=1&page_size=20" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
		</tr>
		<tr>
			<td>advertiser_name</td>
			<td>string</td>
			<td>广告主名称</td>
		</tr>
		<tr>
			<td>corporation_image_name</td>
			<td>string</td>
			<td>品牌名称</td>
		</tr>
		<tr>
			<td>corporation_logo_url</td>
			<td>string</td>
			<td>企业形象LOGO</td>
		</tr>
		<tr>
			<td>industry_id</td>
			<td>integer</td>
			<td>最细一级行业分类（最细有3级）</td>
		</tr>
		<tr>
			<td>corporation_name</td>
			<td>string</td>
			<td>公司名称</td>
		</tr>
		<tr>
			<td>corporation_licence</td>
			<td>string</td>
			<td>ICP号码</td>
		</tr>
		<tr>
			<td>corporate_reg_no</td>
			<td>string</td>
			<td>企业营业执照注册号</td>
		</tr>
		<tr>
			<td>legal_person_id</td>
			<td>string</td>
			<td>身份证号码</td>
		</tr>
		<tr>
			<td>contact_person</td>
			<td>string</td>
			<td>联系人姓名</td>
		</tr>
		<tr>
			<td>contact_person_email</td>
			<td>string</td>
			<td>联系人email</td>
		</tr>
		<tr>
			<td>contact_person_telephone</td>
			<td>string</td>
			<td>联系人座机电话号码</td>
		</tr>
		<tr>
			<td>contact_person_mobile</td>
			<td>string</td>
			<td>联系人手机号码</td>
		</tr>
		<tr>
			<td>individual_qualification</td>
			<td>struct</td>
			<td>身份证明</td>
		</tr>
		<tr>
			<td>certification_image_id</td>
			<td>string</td>
			<td>营业执照/企业资质证明图片id</td>
		</tr>
		<tr>
			<td>certification_image_url</td>
			<td>string</td>
			<td>营业执照/企业资质证明图片URL地址</td>
		</tr>
		<tr>
			<td>website</td>
			<td>string</td>
			<td>推广站点地址</td>
		</tr>
		<tr>
			<td>zip_code</td>
			<td>string</td>
			<td>邮政编码</td>
		</tr>
		<tr>
			<td>company_area_code</td>
			<td>string</td>
			<td>公司所在地区编码</td>
		</tr>
		<tr>
			<td>invoice_area_code</td>
			<td>string</td>
			<td>联系人所在地区编码</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>客户状态</td>
		</tr>
		<tr>
			<td>system_status</td>
			<td>string</td>
			<td>客户系统状态</td>
		</tr>
		<tr>
			<td>fund_status</td>
			<td>string</td>
			<td>资金状态</td>
		</tr>
		<tr>
			<td>virtual_fund_status</td>
			<td>string</td>
			<td>虚拟资金状态</td>
		</tr>
		<tr>
			<td>qualification_image_id_list</td>
			<td>array</td>
			<td>广告特殊资质证明图片ID。最多不超过15个</td>
		</tr>
		<tr>
			<td>qualification_image_url</td>
			<td>array</td>
			<td>行业资质证明图片URL地址。</td>
		</tr>
		<tr>
			<td>ad_qualification_image_id_list</td>
			<td>array</td>
			<td>广告特殊资质证明图片ID。最多不超过15个</td>
		</tr>
		<tr>
			<td>ad_qualification_image_url</td>
			<td>array</td>
			<td>广告特殊资质证明图片URL地址。最多不超过10个</td>
		</tr>
		<tr>
			<td>daily_budget</td>
			<td>integer</td>
			<td>日限额，单位为分</td>
		</tr>
		<tr>
			<td>icp_image_id</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
		</tr>
		<tr>
			<td>icp_image_url</td>
			<td>string</td>
			<td>推广站点ICP备案截图文件URL地址</td>
		</tr>
		<tr>
			<td>audit_message</td>
			<td>string</td>
			<td>审核消息</td>
		</tr>
		<tr>
			<td>outer_advertiser_id</td>
			<td>integer</td>
			<td>外部广告主Id</td>
		</tr>
		<tr>
			<td>submit_draft_enabled</td>
			<td>boolean</td>
			<td>是否允许提交草稿</td>
		</tr>
	</tbody>
</table>
individual_qualification:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>identification_url</td>
			<td>string</td>
			<td>身份证照片URL</td>
		</tr>
		<tr>
			<td>identification_url2</td>
			<td>string</td>
			<td>身份证图片(反面)地址</td>
		</tr>
		<tr>
			<td>photo_url</td>
			<td>string</td>
			<td>半身照URL</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
			"list": [
					{
							"advertiser_id":999,                      // 子客户advertiser_id
							"corporation_name":"hello",             // 子客户公司名称
							"outer_advertiser_id": 9527
							"advertiser_name": "赵州桥责任有限公司",
							"customer_status":'CUSTOMER_STATUS_NORMAL',
							"address": "广东省深圳市南山区xxx路xxx号xxx",
							"fund_status":"FUNDSTATUS_NORMAL",
							"virtual_fund_status":"FUNDSTATUS_NORMAL",
							"daily_budget": 1000,
							...
					},
					{
							...
					}
			],
			"page_info":{
					"total_num":221,                        // 总条数
					"total_page":11,                        // 总页数
					"page_size":10,                         // 每页显示条数
					"page":1,                               // 当前页码
			}
	}
										
```
###2.2.2 合并update_advertiser和set_daily_budget接口
************************************************
	地址：advertiser/update
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>corporation_name</td>
			<td>string</td>
			<td>公司名称</td>
			<td>小于120个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>certification_image_id</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
			<td>32字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>industry_id</td>
			<td>integer</td>
			<td>最细一级行业分类（最细有3级）</td>
			<td>详见 [industry_id | 新行业分类]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>qualification_image_id_list</td>
			<td>array</td>
			<td>广告特殊资质证明图片ID。最多不超过15个</td>
			<td>URL小于255个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>ad_qualification_image_id_list</td>
			<td>array</td>
			<td>广告特殊资质证明图片ID。最多不超过15个</td>
			<td>URL小于255个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>website</td>
			<td>string</td>
			<td>推广站点地址</td>
			<td>URL小于255个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>icp_image_id</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
			<td>32字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>corporation_image_name</td>
			<td>string</td>
			<td>品牌名称</td>
			<td>小于120个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>contact_person_telephone</td>
			<td>string</td>
			<td>联系人座机电话号码</td>
			<td>例如：0755-86013388</td>
			<td>no</td>
		</tr>
		<tr>
			<td>contact_person_mobile</td>
			<td>string</td>
			<td>联系人手机号码</td>
			<td>例如：+8613900000000 或 13900000000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>daily_budget</td>
			<td>integer</td>
			<td>日限额，单位为分</td>
			<td>大于5000，且小于1000000000。账户日限额修改规则:1. 日限额的范围是50元-1000万元（请注意转换为分）2. 日限额的修改幅度不能低于50元（请注意转换为分）3. 日限额的最低金额不能低于账户今日消耗加上50元（请注意转换为分）</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
	注：修改成功后，子客户信息将被重新审核。审核最长需一个工作日。在审核通过前，该子客户的所有广告都将无曝光，审核通过后恢复正常。

请求示例:
```
 curl -d "advertiser_id=999&website=http%3A%2F%2Fqq.com" "http://sandbox.api.e.qq.com/v1.0/advertiser/update?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
		</tr>
		<tr>
			<td>outer_advertiser_id</td>
			<td>integer</td>
			<td>外部广告主Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
				advertiser_id: 123,  // 子客户advertiser_id
				outer_advertiser_id: 39213,
			}
	}
										
```
##2.3 服务商模块

服务商模块描述

###2.3.1 服务商添加子客户
服务商添加子客户描述信息
************************************************
	地址：agency_advertisers/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>corporation_name</td>
			<td>string</td>
			<td>公司名称</td>
			<td>小于120个英文字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>certification_image_id</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
			<td>32字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>industry_id</td>
			<td>integer</td>
			<td>最细一级行业分类（最细有3级）</td>
			<td>详见 [industry_id | 新行业分类]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>outer_industry</td>
			<td>string</td>
			<td>外部行业信息</td>
			<td>外部行业信息</td>
			<td>no</td>
		</tr>
		<tr>
			<td>outer_advertiser_id</td>
			<td>integer</td>
			<td>外部广告主Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>qualification_image_id_list</td>
			<td>array</td>
			<td>广告特殊资质证明图片ID。最多不超过15个</td>
			<td>URL小于255个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>ad_qualification_image_id_list</td>
			<td>array</td>
			<td>广告特殊资质证明图片ID。最多不超过15个</td>
			<td>URL小于255个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>website</td>
			<td>string</td>
			<td>推广站点地址</td>
			<td>URL小于255个英文字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>icp_image_id</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
			<td>32字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>corporation_image_name</td>
			<td>string</td>
			<td>品牌名称</td>
			<td>小于120个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>contact_person_telephone</td>
			<td>string</td>
			<td>联系人座机电话号码</td>
			<td>例如：0755-86013388</td>
			<td>no</td>
		</tr>
		<tr>
			<td>contact_person_mobile</td>
			<td>string</td>
			<td>联系人手机号码</td>
			<td>例如：+8613900000000 或 13900000000</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "outer_advertiser_id=9527&certification_image_url=http%3A%2F%2Fpgdt.gtimg.cn%2Fgdt%2F0%2FCCJFzuAAABT9AOBCifQM 6cF.jpg%2F0%3Fck%3Db78d1e97619d2d5ee244522285c34164& corporation_name=corporation_name&website=http%3A%2F%2Fbadgroup_idu.com& icp_image_url=http%3A%2F%2Fpgdt.gtimg.cn%2Fgdt%2F0%2FCCJFzuAAABT9AOBCifQM6cF.jpg%2F0%3Fck%3Db78d1e97619d2d5ee244522285c34164& qualification_image_url=%7B%22qualificationurl1%22%3A+%22http%3A%2F%2Fpgdt.gtimg.cn%2Fgdt %2F0%2FCAAAMTfAABUFnZQBmOcbXCw.jpg%2F0%3Fck%3D445e5decb66700f4927faf0853b09940%22%7D&industry_id=21474836481" "http://sandbox.api.e.qq.com/v1.0/agency_advertisers/add?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
		</tr>
		<tr>
			<td>outer_advertiser_id</td>
			<td>integer</td>
			<td>外部广告主Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
				advertiser_id: 123,  // 子客户advertiser_id
				outer_advertiser_id: 941322
			}
	}
										
```
##2.4 广告主账户模块

###2.4.1 获取财务账户信息
************************************************
	地址：funds/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
注：CASH、DEVIDE 等key 为账户类型，见[account_type_map | 账户类型定义]

注：fund_status 详见 [fund_status | 资金状态]

请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/funds/get?access_token=<TOKEN>&advertiser_id=999" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>account_type</td>
			<td>string</td>
			<td>账户类型</td>
		</tr>
		<tr>
			<td>balance</td>
			<td>integer</td>
			<td>余额，单位为分</td>
		</tr>
		<tr>
			<td>fund_status</td>
			<td>string</td>
			<td>资金状态</td>
		</tr>
		<tr>
			<td>today_cost</td>
			<td>integer</td>
			<td>今日花费</td>
		</tr>
		<tr>
			<td>effect_funds</td>
			<td>array</td>
			<td>有效资金</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
					"list":[{
							// 现金账户
							"account_type":"CASH",
							"balance":120000, // 余额，单位为分
							"fund_status":"FUNDSTATUS_NORMAL"// 资金状态 正常
							"today_cost":100,   // 今日消费,单位为分
					},
					{
							// 分成账户
							"account_type":"DEVIDE",
							"balance":120000, // 余额，单位为分
							"fund_status":"FUNDSTATUS_NORMAL"// 资金状态 正常
							"today_cost":100,   // 今日消费,单位为分
					},
					{
							// 虚拟金账户
							"account_type":"VIRTUAL",
							"balance":0, // 余额，单位为分
							"fund_status":"FUNDSTATUS_NOT_ENOUGH"，资金状态 余额不足
							"today_cost":100,   // 今日消费,单位为分
					}]
			}
	}
										
```
##2.5 广告主账户模块

###2.5.1 广告主账户日结明细
************************************************
	地址：fund_invoices/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>account_type</td>
			<td>string</td>
			<td>账户类型</td>
			<td>见 [account_type_map | 账户类型定义]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>trade_type</td>
			<td>string</td>
			<td>交易类型</td>
			<td>见 [trade_type | 交易类型]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>date_range</td>
			<td>struct</td>
			<td>时间范围</td>
			<td>日期格式，{"start_date":"2014-03-01","end_date":"2014-04-02"}</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
	注：分页功能和传统分页不同，不是按总条数进行分页，而是按时间范围的总天数进行分页，每天固定返回4条记录（充值、消费、过期和回划），如果请求不提供交易类型，则会返回所有交易类型

请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/fund_invoices/get?access_token=<TOKEN>& advertiser_id=999&account_type=DEVIDE&date_range={"start_date":"2014-03-01","end_date"："2014-04-02"}" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>account_type</td>
			<td>string</td>
			<td>账户类型</td>
		</tr>
		<tr>
			<td>trade_type</td>
			<td>string</td>
			<td>交易类型</td>
		</tr>
		<tr>
			<td>trans_time</td>
			<td>integer</td>
			<td>秒级时间戳</td>
		</tr>
		<tr>
			<td>amount</td>
			<td>integer</td>
			<td>金额</td>
		</tr>
		<tr>
			<td>description</td>
			<td>string</td>
			<td>描述</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
			"list": [               
					{   
							"account_type":"CASH",              // 账户类型
							"trade_type":"RECOVER",             // 交易类型
							"trans_time":1403243242,            // 操作时间戳
							"amount"：4000，                      // 金额（单位分）
							"description":"test",                      // 备注信息
					},
					{
							...
					}
			],
			"page_info":{
					"total_num":221,                        // 总条数
					"total_page":12,                        // 总页数
					"page_size":10,                         // 每页显示条数
					"page":1,                               // 当前页码
			}
	}
										
```
##2.6 广告主账户模块

###2.6.1 获取财务账户流水
************************************************
	地址：fund_transactions/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>account_type</td>
			<td>string</td>
			<td>账户类型</td>
			<td>见 [account_type_map | 账户类型定义]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>date_range</td>
			<td>struct</td>
			<td>时间范围</td>
			<td>日期格式，{"start_date":"2014-03-01","end_date":"2014-04-02"}</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
注：交易类型见 [trade_type | 交易类型定义]

请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/fund_transactions/get?access_token=<TOKEN>&account_type=VIRTUAL&advertiser_id=999& date_range=%7B%22start_date%22%3A%222014-11-01%22%2C%22end_date%22%3A%222014-11-02%22%7D" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>date</td>
			<td>string</td>
			<td>查询时间</td>
		</tr>
		<tr>
			<td>external_bill_no</td>
			<td>string</td>
			<td>外部订单号</td>
		</tr>
		<tr>
			<td>trade_type</td>
			<td>string</td>
			<td>交易类型</td>
		</tr>
		<tr>
			<td>amount</td>
			<td>integer</td>
			<td>金额</td>
		</tr>
		<tr>
			<td>description</td>
			<td>string</td>
			<td>描述</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
					"list": [{
							"date": 2014-03-21,// 交易日期
							"trade_type":'CONSUME',//交易类型
							"amount": 1000000,//交易金额
							"description": "系统扣费",// 备注
							},
							{...}
							],      
					"page_info":{
							"total_num":221,// 流水总条数
							"total_page":11,// 流水总页数
							"page_size":20,// 每页显示条数
							"page":1,// 当前页码
					}
	}
										
```
###2.6.2 transfer和transfer_back合并接口
************************************************
	地址：fund_transactions/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>account_type</td>
			<td>string</td>
			<td>账户类型</td>
			<td>见 [account_type_map | 账户类型定义]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>amount</td>
			<td>integer</td>
			<td>金额</td>
			<td>单位为分</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>transfer_type</td>
			<td>string</td>
			<td>接口类型</td>
			<td>接口类型</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>external_bill_no</td>
			<td>string</td>
			<td>外部订单号</td>
			<td>不超过35字符，需要有调用方标示前缀，且要保证在同一个广告主下唯一，如jdzt-xxx-xxx</td>
			<td>no</td>
		</tr>
		<tr>
			<td>memo</td>
			<td>string</td>
			<td>备注信息</td>
			<td>不超过64个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>expire_date</td>
			<td>string</td>
			<td>开始投放时间点对应的时间戳</td>
			<td>大于等于0，且小于end_time</td>
			<td>no</td>
		</tr>
		<tr>
			<td>outer_advertiser_id</td>
			<td>integer</td>
			<td>外部广告主Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/fund_transactions/add?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>account_type</td>
			<td>string</td>
			<td>账户类型</td>
		</tr>
		<tr>
			<td>amount</td>
			<td>integer</td>
			<td>金额</td>
		</tr>
		<tr>
			<td>external_bill_no</td>
			<td>string</td>
			<td>外部订单号</td>
		</tr>
		<tr>
			<td>trans_time</td>
			<td>integer</td>
			<td>秒级时间戳</td>
		</tr>
		<tr>
			<td>bill_repeated</td>
			<td>string</td>
			<td>是否重复转账</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
					"account_type": CASH,   // 转出账户类型
					"amount": 1000000,      // 转账金额
					"external_bill_no":"gdtxxxxxx",  // 订单号
					"trans_time":12321323,  // 划账时间戳
					"bill_repeated" : 0     // 是否重复转账,1重复，0未重复即正常（订单号重复，只会转账一次）
			}
	}
										
```
##2.7 广告主账户模块

###2.7.1 获取今日消耗
************************************************
	地址：advertiser_cost/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>date</td>
			<td>string</td>
			<td>查询时间</td>
			<td>日期格式，如2014-03-01</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/advertiser_cost/get?access_token=<TOKEN>&advertiser_id=999" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>cost</td>
			<td>integer</td>
			<td>今日花费</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""，
			"data": {
					"today_cost": 1111 // 单位: 分
			}
	}
										
```
##2.8 推广计划模块

###2.8.1 获取推广计划今日消耗
************************************************
	地址：campaigns_cost/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>date</td>
			<td>string</td>
			<td>查询时间</td>
			<td>日期格式，如2014-03-01</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/campaigns_cost/get?access_token=<TOKEN>&advertiser_id=999&campaign_id=22147" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>cost</td>
			<td>integer</td>
			<td>今日消耗</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""，
			"data": {
					"today_cost": 1111 // 单位: 分
			}
	}
										
```
##2.9 推广计划模块

###2.9.1 创建一个推广计划
************************************************
	地址：campaigns/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_name</td>
			<td>string</td>
			<td>推广计划名称</td>
			<td>小于120个英文字符，不可与名下其他推广计划重名</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_type</td>
			<td>string</td>
			<td>推广计划类型</td>
			<td>详见 [campaign_type | 推广计划类型]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>daily_budget</td>
			<td>integer</td>
			<td>日消耗限额，单位为分</td>
			<td>大于1000，且小于400000000</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>total_budget</td>
			<td>integer</td>
			<td>总消耗限额，单位为分</td>
			<td>大于5000，且小于20000000000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>speed_mode</td>
			<td>string</td>
			<td>标准投放类型</td>
			<td>详见 [speed_mode | 标准投放类型]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>pv_demanded</td>
			<td>integer</td>
			<td>每日购买PV量</td>
			<td>大于1000小于42亿</td>
			<td>no</td>
		</tr>
		<tr>
			<td>outer_campaign_id</td>
			<td>integer</td>
			<td>外部推广计划Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>retainability_in_feeds</td>
			<td>string</td>
			<td>沉淀模式</td>
			<td>NO：不支持，YES：支持</td>
			<td>no</td>
		</tr>
		<tr>
			<td>max_impression_include</td>
			<td>integer</td>
			<td>最高曝光频次</td>
			<td>大于等于0、小于等于1000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
			<td></td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "campaign_name=34947448161&daily_budget=20000&advertiser_id=999&campaign_type=CAMPAIGN_TYPE_DISPLAY" "http://sandbox.api.e.qq.com/v1.0/campaigns/add?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
		</tr>
		<tr>
			<td>outer_campaign_id</td>
			<td>integer</td>
			<td>外部推广计划Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""，
			"data":{
				"campaign_id":23456,//推广计划ID
				"outer_campaign_id":19234
			}
	}
										
```
###2.9.2 更新推广计划
************************************************
	地址：campaigns/update
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_name</td>
			<td>string</td>
			<td>推广计划名称</td>
			<td>小于120个英文字符，不可与名下其他推广计划重名</td>
			<td>no</td>
		</tr>
		<tr>
			<td>daily_budget</td>
			<td>integer</td>
			<td>日消耗限额，单位为分</td>
			<td>大于1000，且小于400000000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>total_budget</td>
			<td>integer</td>
			<td>总消耗限额，单位为分</td>
			<td>大于5000，且小于20000000000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>speed_mode</td>
			<td>string</td>
			<td>标准投放类型</td>
			<td>详见 [speed_mode | 标准投放类型]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>retainability_in_feeds</td>
			<td>string</td>
			<td>沉淀模式</td>
			<td>NO：不支持，YES：支持</td>
			<td>no</td>
		</tr>
		<tr>
			<td>max_impression_include</td>
			<td>integer</td>
			<td>最高曝光频次</td>
			<td>大于等于0、小于等于1000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
			<td></td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "status=AD_STATUS_SUSPEND&advertiser_id=999&campaign_id=25986" "http://sandbox.api.e.qq.com/v1.0/campaigns/update?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
		</tr>
		<tr>
			<td>outer_campaign_id</td>
			<td>integer</td>
			<td>外部推广计划Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data":{
				"campaign_id":23456,
				"outer_campaign_id": 49132,
			}   
	}
										
```
###2.9.3 获取推广计划列表
************************************************
	地址：campaigns/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>filter</td>
			<td>array</td>
			<td>过滤条件</td>
			<td>若此字段不传，或传空则视为无限制条件。例{"configured_status":"AD_STATUS_NORMAL"}, 可选过滤字段：configured_status，可取值：AD_STATUS_NORMAL、AD_STATUS_SUSPEND，system_status，可取值：AD_STATUS_NORMAL、AD_STATUS_PENDING、AD_STATUS_DENIED，adgroup_name，campaign_id。</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/campaigns/get?access_token=<TOKEN>& where=%7B%22campaign_name%22%3A%2234947448161%22%7D&advertiser_id=999" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
		</tr>
		<tr>
			<td>campaign_name</td>
			<td>string</td>
			<td>推广计划名称</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
		</tr>
		<tr>
			<td>campaign_type</td>
			<td>string</td>
			<td>推广计划类型</td>
		</tr>
		<tr>
			<td>outer_campaign_id</td>
			<td>integer</td>
			<td>外部推广计划Id</td>
		</tr>
		<tr>
			<td>daily_budget</td>
			<td>integer</td>
			<td>日消耗限额，单位为分</td>
		</tr>
		<tr>
			<td>total_budget</td>
			<td>integer</td>
			<td>总消耗限额，单位为分</td>
		</tr>
		<tr>
			<td>budget_limit_date</td>
			<td>integer</td>
			<td>日限额到达日期</td>
		</tr>
		<tr>
			<td>created_time</td>
			<td>integer</td>
			<td>创建时间</td>
		</tr>
		<tr>
			<td>last_modified_time</td>
			<td>integer</td>
			<td>最后修改时间</td>
		</tr>
		<tr>
			<td>completed_time</td>
			<td>integer</td>
			<td>合约完成时间</td>
		</tr>
		<tr>
			<td>pv_demanded</td>
			<td>integer</td>
			<td>每日购买PV量</td>
		</tr>
		<tr>
			<td>speed_mode</td>
			<td>string</td>
			<td>标准投放类型</td>
		</tr>
		<tr>
			<td>retainability_in_feeds</td>
			<td>string</td>
			<td>沉淀模式</td>
		</tr>
		<tr>
			<td>max_impression_include</td>
			<td>integer</td>
			<td>最高曝光频次</td>
		</tr>
		<tr>
			<td>begin_date</td>
			<td>string</td>
			<td>开始投放时间点对应的时间戳</td>
		</tr>
		<tr>
			<td>end_date</td>
			<td>string</td>
			<td>结束投放时间点对应的时间戳点对应的时间戳</td>
		</tr>
		<tr>
			<td>site_set</td>
			<td>array</td>
			<td>投放站点集合</td>
		</tr>
		<tr>
			<td>time_series</td>
			<td>string</td>
			<td>投放时间段，格式为48 * 7位由0和1组成的字符串，也就是以半个小时为最小粒度，0为不投放，1为投放</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
		"ret":0,
		"msg":""，
		"data":{
			"list": [{
				"campaign_id":23456,
				"campaign_name":"xxx推广计划",
				"budget_limit_date":20140102，//日限额到达日期
				"status":"AD_STATUS_NORMAL",// 启用
				"outer_campaign_id": 54951,
			},
			{...}
		],
		"page_info":{
			"total_num":221,// 推广计划条数
			"total_page":11,// 推广计划总页数
			"page_size":10,// 每页显示条数
			"page":1,// 当前页码
		}
	}
										
```
###2.9.4 删除推广计划
************************************************
	地址：campaigns/delete
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
	注：正常投放中的推广计划不能删除，需要先做暂停操作，然后再进行删除。推广计划删除操作，除了删除推广计划本身之外，也会同时删除推广计划下的所有广告，且此操作不可逆，请谨慎调用！

请求示例:
```
 curl -d "campaign_id=23456" "http://sandbox.api.e.qq.com/v1.0/campaigns/delete? access_token=NTEwOTQsNTEwOTQsMTQxNjI4NTU1NywzN2I4OTY5M2RlNmNmOGQ4MmNhZWI0Mjk5MDcyMGY1YWFlYjBiY2Ix" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data":{
				"campaign_id":23456,
				"outer_campaign_id": 19931,
			}
	}
										
```
##2.10 广告组模块

###2.10.1 创建一个广告组
************************************************
	地址：adgroups/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>adgroup_name</td>
			<td>string</td>
			<td>广告组名称</td>
			<td>小于120个英文字符，同一账户下名称不允许重复。</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>bid_type</td>
			<td>string</td>
			<td>扣费方式，如CPD、CPM</td>
			<td>详见 [bid_type | 扣费方式]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>bid_amount</td>
			<td>integer</td>
			<td>广告出价，单位为分</td>
			<td>广告出价，单位为分</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>begin_date</td>
			<td>string</td>
			<td>开始投放时间点对应的时间戳</td>
			<td>大于等于0，且小于end_time</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>end_date</td>
			<td>string</td>
			<td>结束投放时间点对应的时间戳点对应的时间戳</td>
			<td>大于等于今天，且大于begin_time</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>site_set</td>
			<td>array</td>
			<td>投放站点集合</td>
			<td>当前仅支持单站点，取值详见 [site_set_definition | 投放站点集合]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>outer_adgroup_id</td>
			<td>integer</td>
			<td>外部广告Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>time_series</td>
			<td>string</td>
			<td>投放时间段，格式为48 * 7位由0和1组成的字符串，也就是以半个小时为最小粒度，0为不投放，1为投放</td>
			<td>等于48*7位字符串，且都是0或1，不传此字段则视为全时段投放</td>
			<td>no</td>
		</tr>
		<tr>
			<td>product_type</td>
			<td>string</td>
			<td>标的物类型</td>
			<td>详见 [product_type | 标的物类型]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_refs_id</td>
			<td>string</td>
			<td>标的物Id</td>
			<td>小于128个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>subordinate_product_refs_id</td>
			<td>string</td>
			<td>子标的物id（渠道包id）</td>
			<td>小于128个英文字符，从开放平台api获取</td>
			<td>no</td>
		</tr>
		<tr>
			<td>creative_selection_type</td>
			<td>string</td>
			<td>素材播放模式</td>
			<td>详见 [creative_selection_type | 素材播放模式]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>customized_category</td>
			<td>string</td>
			<td>自定义分类，关键词分割，如，本地生活——餐饮</td>
			<td>小于等于200个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>min_impression_include</td>
			<td>integer</td>
			<td>最低曝光频次</td>
			<td>大于等于0、小于等于1000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>max_impression_include</td>
			<td>integer</td>
			<td>最高曝光频次</td>
			<td>大于等于0、小于等于1000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>click_tracking_url</td>
			<td>string</td>
			<td>监控链接</td>
			<td>小于1024个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>creative_combination_type</td>
			<td>string</td>
			<td>广告类型，支持普通广告、集装箱广告和动态创意广告</td>
			<td>详见 [creative_combination_type | 广告类型]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>dynamic_creative_recommend_type</td>
			<td>integer</td>
			<td>产品推荐方式</td>
			<td>允许值可通过接口utility/get_dynamic_right_info获取</td>
			<td>no</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>total_budget</td>
			<td>integer</td>
			<td>总消耗限额，单位为分</td>
			<td>大于5000，且小于20000000000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>payment_type</td>
			<td>string</td>
			<td>付款类型</td>
			<td>允许值: 预付费，实时扣费两种</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "outer_adgroup_id=95338&advertiser_id=51489&creative_selection_type=CREATIVE_SELECTION_TYPE_AUTO_OPTIMIZED& bid=9000&start_date=2014-11-18&site_set=%5B%22SITE_SET_QQCLIENT%22%5D& adgroup_name=2814753028&product_type=PRODUCT_TYPE_DP_TUAN&product_refs_id=willowdkjejfldkjf& campaign_id=22147&&targeting_id=27576&bid_type=COSTTYPE_CPC& end_date=2014-11-20&cate_root=1" "http://sandbox.api.e.qq.com/v1.0/adgroups/add?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
		</tr>
		<tr>
			<td>outer_adgroup_id</td>
			<td>integer</td>
			<td>外部广告Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""，
			"data":{
				"adgroup_id":123456,//广告组ID
				"outer_adgroup_id": 429931,
			}
	}
										
```
###2.10.2 更新广告组
************************************************
	地址：adgroups/update
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
			<td></td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>adgroup_name</td>
			<td>string</td>
			<td>广告组名称</td>
			<td>小于120个英文字符，同一账户下名称不允许重复。</td>
			<td>no</td>
		</tr>
		<tr>
			<td>bid_amount</td>
			<td>integer</td>
			<td>广告出价，单位为分</td>
			<td>广告出价，单位为分</td>
			<td>no</td>
		</tr>
		<tr>
			<td>begin_date</td>
			<td>string</td>
			<td>开始投放时间点对应的时间戳</td>
			<td>大于等于0，且小于end_time</td>
			<td>no</td>
		</tr>
		<tr>
			<td>end_date</td>
			<td>string</td>
			<td>结束投放时间点对应的时间戳点对应的时间戳</td>
			<td>大于等于今天，且大于begin_time</td>
			<td>no</td>
		</tr>
		<tr>
			<td>time_series</td>
			<td>string</td>
			<td>投放时间段，格式为48 * 7位由0和1组成的字符串，也就是以半个小时为最小粒度，0为不投放，1为投放</td>
			<td>等于48*7位字符串，且都是0或1，不传此字段则视为全时段投放</td>
			<td>no</td>
		</tr>
		<tr>
			<td>creative_selection_type</td>
			<td>string</td>
			<td>素材播放模式</td>
			<td>详见 [creative_selection_type | 素材播放模式]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>customized_category</td>
			<td>string</td>
			<td>自定义分类，关键词分割，如，本地生活——餐饮</td>
			<td>小于等于200个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>min_impression_include</td>
			<td>integer</td>
			<td>最低曝光频次</td>
			<td>大于等于0、小于等于1000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>max_impression_include</td>
			<td>integer</td>
			<td>最高曝光频次</td>
			<td>大于等于0、小于等于1000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>click_tracking_url</td>
			<td>string</td>
			<td>监控链接</td>
			<td>小于1024个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>subordinate_product_refs_id</td>
			<td>string</td>
			<td>子标的物id（渠道包id）</td>
			<td>小于128个英文字符，从开放平台api获取</td>
			<td>no</td>
		</tr>
		<tr>
			<td>dynamic_creative_recommend_type</td>
			<td>integer</td>
			<td>产品推荐方式</td>
			<td>允许值可通过接口utility/get_dynamic_right_info获取</td>
			<td>no</td>
		</tr>
		<tr>
			<td>total_budget</td>
			<td>integer</td>
			<td>总消耗限额，单位为分</td>
			<td>大于5000，且小于20000000000</td>
			<td>no</td>
		</tr>
		<tr>
			<td>payment_type</td>
			<td>string</td>
			<td>付款类型</td>
			<td>允许值: 预付费，实时扣费两种</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
	注：当需要修改状态值时，无法修改当前状态值非AD_STATUS_NORMAL,AD_STATUS_SUSPEND的广告组，且只允许修改为AD_STATUS_NORMAL,AD_STATUS_SUSPEND其中之一

请求示例:
```
 curl -d "adgroup_id=30304&product_type=PRODUCT_TYPE_DP_COUPON&advertiser_id=999" "http://sandbox.api.e.qq.com/v1.0/adgroups/update?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
		</tr>
		<tr>
			<td>outer_adgroup_id</td>
			<td>integer</td>
			<td>外部广告Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data":{
				"adgroup_id":123456,
				"outer_adgroup_id": 91239,
			}
	}
										
```
###2.10.3 删除一条广告组
************************************************
	地址：adgroups/delete
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
			<td></td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "adgroup_id=47459&advertiser_id=999" "http://sandbox.api.e.qq.com/v1.0/adgroups/delete?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
		</tr>
		<tr>
			<td>outer_adgroup_id</td>
			<td>integer</td>
			<td>外部广告Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""
			"data":{
				"adgroup_id":123456,//广告组ID
				"outer_adgroup_id": 0,
			}
	}
										
```
###2.10.4 获取广告组列表
************************************************
	地址：adgroups/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>filter</td>
			<td>array</td>
			<td>过滤条件</td>
			<td>若此字段不传，或传空则视为无限制条件。例{"configured_status":"AD_STATUS_NORMAL"}, 可选过滤字段：configured_status，可取值：AD_STATUS_NORMAL、AD_STATUS_SUSPEND，system_status，可取值：AD_STATUS_NORMAL、AD_STATUS_PENDING、AD_STATUS_DENIED，adgroup_name，campaign_id。</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
	注：当同时传where和filter参数时，优先取filter

请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/adgroups/get?access_token=<TOKEN>&advertiser_id=999" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
		</tr>
		<tr>
			<td>subordinate_product_refs_id</td>
			<td>string</td>
			<td>子标的物id（渠道包id）</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
		</tr>
		<tr>
			<td>system_status</td>
			<td>string</td>
			<td>系统状态</td>
		</tr>
		<tr>
			<td>adgroup_name</td>
			<td>string</td>
			<td>广告组名称</td>
		</tr>
		<tr>
			<td>bid_type</td>
			<td>string</td>
			<td>扣费方式，如CPD、CPM</td>
		</tr>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
		</tr>
		<tr>
			<td>outer_adgroup_id</td>
			<td>integer</td>
			<td>外部广告Id</td>
		</tr>
		<tr>
			<td>begin_date</td>
			<td>string</td>
			<td>开始投放时间点对应的时间戳</td>
		</tr>
		<tr>
			<td>end_date</td>
			<td>string</td>
			<td>结束投放时间点对应的时间戳点对应的时间戳</td>
		</tr>
		<tr>
			<td>time_series</td>
			<td>string</td>
			<td>投放时间段，格式为48 * 7位由0和1组成的字符串，也就是以半个小时为最小粒度，0为不投放，1为投放</td>
		</tr>
		<tr>
			<td>site_set</td>
			<td>array</td>
			<td>投放站点集合</td>
		</tr>
		<tr>
			<td>creative_selection_type</td>
			<td>string</td>
			<td>素材播放模式</td>
		</tr>
		<tr>
			<td>product_type</td>
			<td>string</td>
			<td>标的物类型</td>
		</tr>
		<tr>
			<td>product_refs_id</td>
			<td>string</td>
			<td>标的物Id</td>
		</tr>
		<tr>
			<td>audit_message</td>
			<td>string</td>
			<td>审核消息</td>
		</tr>
		<tr>
			<td>customized_category</td>
			<td>string</td>
			<td>自定义分类，关键词分割，如，本地生活——餐饮</td>
		</tr>
		<tr>
			<td>meta_class</td>
			<td>integer</td>
			<td>广告类目</td>
		</tr>
		<tr>
			<td>cate_root</td>
			<td>integer</td>
			<td>一级分类</td>
		</tr>
		<tr>
			<td>bid_amount</td>
			<td>integer</td>
			<td>广告出价，单位为分</td>
		</tr>
		<tr>
			<td>click_tracking_url</td>
			<td>string</td>
			<td>监控链接</td>
		</tr>
		<tr>
			<td>creative_combination_type</td>
			<td>string</td>
			<td>广告类型，支持普通广告、集装箱广告和动态创意广告</td>
		</tr>
		<tr>
			<td>dynamic_creative_recommend_type</td>
			<td>integer</td>
			<td>产品推荐方式</td>
		</tr>
		<tr>
			<td>min_impression_include</td>
			<td>integer</td>
			<td>最低曝光频次</td>
		</tr>
		<tr>
			<td>max_impression_include</td>
			<td>integer</td>
			<td>最高曝光频次</td>
		</tr>
		<tr>
			<td>created_time</td>
			<td>integer</td>
			<td>创建时间</td>
		</tr>
		<tr>
			<td>last_modified_time</td>
			<td>integer</td>
			<td>最后修改时间</td>
		</tr>
		<tr>
			<td>total_budget</td>
			<td>integer</td>
			<td>总消耗限额，单位为分</td>
		</tr>
		<tr>
			<td>completed_time</td>
			<td>integer</td>
			<td>广告总预算达成时间</td>
		</tr>
		<tr>
			<td>payment_type</td>
			<td>string</td>
			<td>付款类型</td>
		</tr>
		<tr>
			<td>destination_url</td>
			<td>string</td>
			<td>素材目标url</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data":{
					"list":[{
							"adgroup_id":1,
							"adgroup_name":"点击即可免费获赠《英语电子宝典》 ",
							"status": "AD_STATUS_NORMAL",
							"audit_message": "" // 该广告的审核信息，状态为不通过时才有值
							"outer_adgroup_id": 51238,
						},
						{
							"adgroup_id":2,
							"adgroup_name":"免费获赠《英语电子宝典》",
							"status": "AD_STATUS_NORMAL",
							"audit_message": "" // 该广告的审核信息，状态为不通过时才有值
							"outer_adgroup_id": 51350,
						},
						...
					],
					"page_info":{
						"total_num":221,//总广告条数
						"total_page":11,//总页数
						"page_size":10,//每页显示条数
						"page":1,//当前页码
					}
			}
	}
										
```
##2.11 广告素材模块

###2.11.1 创建一条素材
************************************************
	地址：creatives/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
			<td></td>
			<td>yes</td>
		</tr>
		<tr>
			<td>creative_name</td>
			<td>string</td>
			<td>素材名称</td>
			<td>小于120个英文字符，同一账户下名称不允许重复。</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>creative_template_id</td>
			<td>integer</td>
			<td>素材规格Id</td>
			<td>详见 [creative_template_id | 素材规格Id]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>creative_elements</td>
			<td>string</td>
			<td>素材元素</td>
			<td>不超过16384个字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>destination_url</td>
			<td>string</td>
			<td>素材目标url</td>
			<td>小于1023个英文字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>impression_tracking_url</td>
			<td>string</td>
			<td>曝光监控地址</td>
			<td>小于1023个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>click_tracking_url</td>
			<td>string</td>
			<td>监控链接</td>
			<td>小于1024个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>dynamic_creative_template_id</td>
			<td>integer</td>
			<td>动态创意模板ID（仅动态创意特性允许使用）</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>dynamic_creative_material_label</td>
			<td>string</td>
			<td>动态创意模板物料标签（仅动态创意特性允许使用）</td>
			<td>小于120个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>outer_creative_id</td>
			<td>integer</td>
			<td>外部广告素材Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
			<td></td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "outer_creative_id=74138&creative_template_id=25&advertiser_id=51489&campaign_id=22147& creative_name=5291481035&adgroup_id=30304& creative_elements=%7B%22title%22:%22title%20text%22,%22content%22:%22content%20text%22,%22top_icon%22:%22top_icon%22,%22img%22:%22IMG_ID%22,%22button_text%22:%22button%20text%22,%22bottom_text%22:%22bottom%20text%22,%22activity_type%22:%22BIRTHDAY_ACTIVITY_PAGE_CARD%22,%22text_img%22:%5B%7B%22title%22:%22text%20img%20title%22,%22img%22:%22text%20img%20img%22%7D,%7B%22title%22:%22text%20img%20title%22,%22img%22:%22text%20img%20img%22%7D,%7B%22title%22:%22text%20img%20title%22,%22img%22:%22text%20img%20img%22%7D%5D,%22activity_type2%22:%7B%22text_1%22:%22text%201%22%7D%7D%0A%0A" "http://sandbox.api.e.qq.com/v1.0/creatives/add?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
		</tr>
		<tr>
			<td>outer_creative_id</td>
			<td>integer</td>
			<td>外部广告素材Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""，
			"data":{
				"creative_id":123456,
				"outer_creative_id": 74138,
			}
	}
										
```
###2.11.2 更新素材
************************************************
	地址：creatives/update
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>creative_name</td>
			<td>string</td>
			<td>素材名称</td>
			<td>小于120个英文字符，同一账户下名称不允许重复。</td>
			<td>no</td>
		</tr>
		<tr>
			<td>creative_template_id</td>
			<td>integer</td>
			<td>素材规格Id</td>
			<td>详见 [creative_template_id | 素材规格Id]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>creative_elements</td>
			<td>string</td>
			<td>素材元素</td>
			<td>不超过16384个字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>destination_url</td>
			<td>string</td>
			<td>素材目标url</td>
			<td>小于1023个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>impression_tracking_url</td>
			<td>string</td>
			<td>曝光监控地址</td>
			<td>小于1023个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>click_tracking_url</td>
			<td>string</td>
			<td>监控链接</td>
			<td>小于1024个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>dynamic_creative_template_id</td>
			<td>integer</td>
			<td>动态创意模板ID（仅动态创意特性允许使用）</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>dynamic_creative_material_label</td>
			<td>string</td>
			<td>动态创意模板物料标签（仅动态创意特性允许使用）</td>
			<td>小于120个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
			<td></td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "creative_template_id=24&advertiser_id=999& creative_elements=%7B%22title%22:%22title%20text%22,%22content%22:%22content%20text%22,%22top_icon%22:%22top_icon%22,%22img%22:%22IMG_ID%22,%22button_text%22:%22button%20text%22,%22bottom_text%22:%22bottom%20text%22,%22activity_type%22:%22BIRTHDAY_ACTIVITY_PAGE_CARD%22,%22text_img%22:%5B%7B%22title%22:%22text%20img%20title%22,%22img%22:%22text%20img%20img%22%7D,%7B%22title%22:%22text%20img%20title%22,%22img%22:%22text%20img%20img%22%7D,%7B%22title%22:%22text%20img%20title%22,%22img%22:%22text%20img%20img%22%7D%5D,%22activity_type2%22:%7B%22text_1%22:%22text%201%22%7D%7D%0A%0A& creative_id=28498" "http://sandbox.api.e.qq.com/v1.0/creatives/update?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
		</tr>
		<tr>
			<td>outer_creative_id</td>
			<td>integer</td>
			<td>外部广告素材Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data":{
				"creative_id":123456,
				"outer_creative_id": 0,
			}
	}
										
```
###2.11.3 删除一条素材
************************************************
	地址：creatives/delete
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "creative_id=39140&advertiser_id=999" "http://sandbox.api.e.qq.com/v1.0/creatives/delete?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
		</tr>
		<tr>
			<td>outer_creative_id</td>
			<td>integer</td>
			<td>外部广告素材Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data":{
				"creative_id":123456,
				"outer_creative_id": 0
			}
	}
										
```
###2.11.4 获取素材列表
************************************************
	地址：creatives/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>filter</td>
			<td>array</td>
			<td>过滤条件</td>
			<td>若此字段不传，或传空则视为无限制条件。例{"configured_status":"AD_STATUS_NORMAL"}, 可选过滤字段：configured_status，可取值：AD_STATUS_NORMAL、AD_STATUS_SUSPEND，system_status，可取值：AD_STATUS_NORMAL、AD_STATUS_PENDING、AD_STATUS_DENIED，adgroup_name，campaign_id。</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/creatives/get?access_token=<TOKEN>&where=%7B%22adgroup_id%22%3A2885206%7D&advertiser_id=999" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
		</tr>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
		</tr>
		<tr>
			<td>outer_creative_id</td>
			<td>integer</td>
			<td>外部广告素材Id</td>
		</tr>
		<tr>
			<td>creative_name</td>
			<td>string</td>
			<td>素材名称</td>
		</tr>
		<tr>
			<td>configured_status</td>
			<td>string</td>
			<td>用户状态</td>
		</tr>
		<tr>
			<td>system_status</td>
			<td>string</td>
			<td>系统状态</td>
		</tr>
		<tr>
			<td>creative_template_id</td>
			<td>integer</td>
			<td>素材规格Id</td>
		</tr>
		<tr>
			<td>creative_elements</td>
			<td>string</td>
			<td>素材元素</td>
		</tr>
		<tr>
			<td>destination_url</td>
			<td>string</td>
			<td>素材目标url</td>
		</tr>
		<tr>
			<td>dynamic_creative_template_id</td>
			<td>integer</td>
			<td>动态创意模板ID（仅动态创意特性允许使用）</td>
		</tr>
		<tr>
			<td>dynamic_creative_material_label</td>
			<td>string</td>
			<td>动态创意模板物料标签（仅动态创意特性允许使用）</td>
		</tr>
		<tr>
			<td>impression_tracking_url</td>
			<td>string</td>
			<td>曝光监控地址</td>
		</tr>
		<tr>
			<td>click_tracking_url</td>
			<td>string</td>
			<td>监控链接</td>
		</tr>
		<tr>
			<td>created_time</td>
			<td>integer</td>
			<td>创建时间</td>
		</tr>
		<tr>
			<td>last_modified_time</td>
			<td>integer</td>
			<td>最后修改时间</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
		"ret":0,
		"msg":"",
		"data":{
			"list":[{
				"adgroup_id": 123456,
				"adgroup_name": "app推广广告素材",
				"status": "AD_STATUS_NORMAL",
				"outer_creative_id": 74138,
			},
			{
				"adgroup_id": 123457,
				"adgroup_name": "app推广广告素材2",
				"status": "AD_STATUS_NORMAL",
				"outer_creative_id": 74159,
			}],
		},
		"page_info":{
			"total_num":221,// 推广计划条数
			"total_page":11,// 推广计划总页数
			"page_size":10,// 每页显示条数
			"page":1,// 当前页码
		}
	}
										
```
##2.12 创意交换平台

###2.12.1 制作创意
************************************************
	地址：images_exchange/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td></td>
			<td>yes</td>
		</tr>
		<tr>
			<td>crt_sizes</td>
			<td>array</td>
			<td>广点通创意规格ID集合</td>
			<td></td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting</td>
			<td>struct</td>
			<td>广告信息</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>agency_id</td>
			<td>integer</td>
			<td>广点通侧平台ID</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>agency</td>
			<td>struct</td>
			<td>平台方信息</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>shop</td>
			<td>struct</td>
			<td>平台方信息</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>products</td>
			<td>array</td>
			<td>商品信息，可能有多个</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>advertisement</td>
			<td>struct</td>
			<td>广告信息</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>preference</td>
			<td>struct</td>
			<td>创意制作配置项</td>
			<td></td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/images_exchange/add" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>total_count</td>
			<td>integer</td>
			<td>生成创意总数</td>
		</tr>
		<tr>
			<td>valid_count</td>
			<td>integer</td>
			<td>有效创意数</td>
		</tr>
		<tr>
			<td>creatives</td>
			<td>array</td>
			<td>创意集合</td>
		</tr>
		<tr>
			<td>req_no</td>
			<td>string</td>
			<td>用于跟踪请求处理过程的标识</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
        code: 0,                                                    // [必填] 错误码
        data: {                                                     // [必填] 实际数据节点
            total_count: 123,                                       // [必填] 生成创意总数
            valid_count: 120,                                       // [必填] 有效创意数
            creatives: [                                            // [必填] 创意集合
                {
                    creative_id: 123,                               // [必填] 创意 ID
                    template_id: 123,                               // [可选] 模板 ID
                    crt_size: 12,                                   // [必填] 广点通创意规格 ID
                    provider_id: 123,                               // [必填] 供应商 ID
                    provider: {                                     // [必填] 供应商侧元数据
                        creative_id: "the-creative-id",             // [必填] 创意 ID
                        template_id: "the-template-id"              // [可选] 模板 ID
                    },
                    content: {                                      // [必填] 创意内容
                        title: "the-title",                         // [可选] 广告标题
                        desc: "the-desc",                           // [可选] 广告描述信息
                        image_url: "http://the-url-of-image",       // [可选] 图片地址
                        image_resource_key: "venus:ctx-work:v1:806410-2814-612308-1",   // [可选] 图片资源标识
                        image_md5: "md5-of-image",                  // [可选] 图片 MD5
                        image2_url: "http://the-url-of-image2",     // [可选] 图片 2 地址
                        image2_resource_key: "venus:ctx-work:v1:806410-2980-612307-1",  // [可选] 图片 2 资源标识
                        image2_md5: "md5-of-image2"                 // [可选] 图片 2 MD5
                    }
                },
                ...
            ],
            req_no: "6325d8c4-9f20-4d50-bf8c-5b951b7d5f38"          // [可选] 用于跟踪请求处理过程的标识
        }
    }
										
```
##2.13 创意交换平台

###2.13.1 查询可用创意模板
************************************************
	地址：images_exchange_templates/get
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td></td>
			<td>yes</td>
		</tr>
		<tr>
			<td>crt_size</td>
			<td>integer</td>
			<td>广点通创意规格ID</td>
			<td></td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/images_exchange_templates/get" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>templates</td>
			<td>array</td>
			<td>创意模板集合</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
        code: 0,                                                    // [必填] 错误码
        data: {                                                     // [必填] 实际数据节点
            templates: [                                            // [必填] 创意模板集合
                {
                    template_id: 123,                               // [必填] 模板 ID
                    provider_id: 123,                               // [必填] 供应商标识
                    provider: {                                     // [必填] 供应商侧元数据
                        template_id: "the-template-id"              // [必填] 供应商模板 ID
                    },
                    content: {                                      // [必填] 模板内容
                        title: "the-title",                         // [必填] 模板标题
                        desc: "the-desc",                           // [可选] 模板描述信息
                        thumbnail_url: "http://the-url-of-thumb",   // [必填] 模板缩略图地址
                        version: 1,                                 // [可选] 版本号
                        timestamp: 1234567890                       // [可选] 更新时间戳，秒
                    }
                },
                ...
            ]
        }
    }
										
```
##2.14 标的物模块

###2.14.1 创建一个标的物
************************************************
	地址：products/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_refs_id</td>
			<td>string</td>
			<td>标的物Id</td>
			<td>小于128个英文字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_name</td>
			<td>string</td>
			<td>标的物名称</td>
			<td>小于255个英文字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_type</td>
			<td>string</td>
			<td>标的物类型</td>
			<td>详见 [product_type | 标的物类型]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_info</td>
			<td>struct</td>
			<td>标的物详细信息</td>
			<td>详见 [ec_info | 京东、拍拍店铺、标的物]</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "advertiser_id=9999&product_name=my_product&product_refs_id=C3C2X&product_type=product_type&product_info={"ec_info":{"product_price":18900,"product_meta_class":241588}}" "http://sandbox.api.e.qq.com/v1.0/products/add?access_token=<TOKEN>" 
```
返回示例：
```

	{
			"ret":0,
			"msg":""
	}
										
```
###2.14.2 更新标的物信息
************************************************
	地址：products/update
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_refs_id</td>
			<td>string</td>
			<td>标的物Id</td>
			<td>小于128个英文字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_name</td>
			<td>string</td>
			<td>标的物名称</td>
			<td>小于255个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>product_type</td>
			<td>string</td>
			<td>标的物类型</td>
			<td>详见 [product_type | 标的物类型]</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_info</td>
			<td>struct</td>
			<td>标的物详细信息</td>
			<td>详见 [ec_info | 京东、拍拍店铺、标的物]</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "advertiser_id=9999&product_refs_id=C2C3X" "http://sandbox.api.e.qq.com/v1.0/products/update?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>product_refs_id</td>
			<td>string</td>
			<td>标的物Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
	}
										
```
###2.14.3 合并read和utility/get_subordinate_product_list接口
************************************************
	地址：products/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_refs_id</td>
			<td>string</td>
			<td>标的物Id</td>
			<td>小于128个英文字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>product_type</td>
			<td>string</td>
			<td>标的物类型</td>
			<td>详见 [product_type | 标的物类型]</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/products/get?access_token=<TOKEN>&advertiser_id=9999&product_refs_id=C3C2X" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>product_refs_id</td>
			<td>string</td>
			<td>标的物Id</td>
		</tr>
		<tr>
			<td>product_name</td>
			<td>string</td>
			<td>标的物名称</td>
		</tr>
		<tr>
			<td>product_type</td>
			<td>string</td>
			<td>标的物类型</td>
		</tr>
		<tr>
			<td>product_info</td>
			<td>struct</td>
			<td>标的物详细信息</td>
		</tr>
		<tr>
			<td>created_time</td>
			<td>integer</td>
			<td>创建时间</td>
		</tr>
		<tr>
			<td>last_modified_time</td>
			<td>integer</td>
			<td>最后修改时间</td>
		</tr>
		<tr>
			<td>subordinate_product_list</td>
			<td>array</td>
			<td>子标的物详细信息列表</td>
		</tr>
	</tbody>
</table>
product_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>product_type_jd_item</td>
			<td>struct</td>
			<td>电商标的物信息</td>
		</tr>
		<tr>
			<td>product_type_jd_shop</td>
			<td>struct</td>
			<td>电商标的物信息</td>
		</tr>
		<tr>
			<td>product_type_apple_app_store</td>
			<td>struct</td>
			<td>ios应用信息</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data":{
					"product_name": "my_product",
					"product_type": "PROUDCTTYPE_JD_ITEM",
					"product_refs_id": "C3C2X",
					"product_info": {
						"jd_item": {
							"product_price": 18900,
							"product_meta_class": 241588
						}
					}
			}
	}
										
```
##2.15 定向模块

###2.15.1 创建一个定向
************************************************
	地址：targetings/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting_name</td>
			<td>string</td>
			<td>定向名称</td>
			<td>小于等于120个英文字符，同一账户下名称不允许重复。</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>description</td>
			<td>string</td>
			<td>定向描述</td>
			<td>小于250个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>ui_visibility</td>
			<td>string</td>
			<td>定向包类型</td>
			<td>详见 [ui_visibility | 定向包类型]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>targeting_setting</td>
			<td>struct</td>
			<td>定向详细设置</td>
			<td>存放所有定向条件</td>
			<td>no</td>
		</tr>
		<tr>
			<td>outer_targeting_id</td>
			<td>integer</td>
			<td>外部广告定向Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d 'advertiser_id=$advertiser_id&outer_targeting_id=$outer_targeting_id&targeting_id=$targeting_id&gender=["FEMALE"]& shopping_behavior_jd={"object_type":"OBJECT_TYPE_CATEGORY","object_id_list":[2,101], "time_window":"TIME_WINDOW_MEDIUM","act_id_list":["ACT_SHOPPING"]}' 'http://sandbox.api.e.qq.com/v1.0/targetings/add?access_token=<TOKEN>' 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
		</tr>
		<tr>
			<td>outer_targeting_id</td>
			<td>integer</td>
			<td>外部广告定向Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data": {
				"targeting_id": 1,
				"outer_targeting_id": 0
			}
	}
										
```
###2.15.2 更新定向信息
************************************************
	地址：targetings/update
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting_name</td>
			<td>string</td>
			<td>定向名称</td>
			<td>小于等于120个英文字符，同一账户下名称不允许重复。</td>
			<td>no</td>
		</tr>
		<tr>
			<td>description</td>
			<td>string</td>
			<td>定向描述</td>
			<td>小于250个英文字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>targeting_setting</td>
			<td>struct</td>
			<td>定向详细设置</td>
			<td>存放所有定向条件</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "advertiser_id=999&region=%5B110000%2C120000%2C994840%2C994670%2C994332%2C140000%5D& gender=%5B%22MALE%22%5D&age=%5B%225%7E60%22%5D&targeting_id=27576&location=%5B%5D" "http://sandbox.api.e.qq.com/v1.0/targetings/update?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
		</tr>
		<tr>
			<td>outer_targeting_id</td>
			<td>integer</td>
			<td>外部广告定向Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""，
			"data":{
				"targeting_id":123456,
				"outer_targeting_id": 0,
			}
	}
										
```
###2.15.3 删除定向
************************************************
	地址：targetings/delete
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "advertiser_id=999&targeting_id=27576" "http://sandbox.api.e.qq.com/v1.0/targetings/delete?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
		</tr>
		<tr>
			<td>outer_targeting_id</td>
			<td>integer</td>
			<td>外部广告定向Id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""，
			"data":{
				"targeting_id":123,
				"outer_targeting_id": 0
			}
	}
										
```
###2.15.4 获取定向列表
************************************************
	地址：targetings/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
			<td>小于2^63</td>
			<td>no</td>
		</tr>
		<tr>
			<td>filter</td>
			<td>array</td>
			<td>过滤条件</td>
			<td>若此字段不传，或传空则视为无限制条件。例{"configured_status":"AD_STATUS_NORMAL"}, 可选过滤字段：configured_status，可取值：AD_STATUS_NORMAL、AD_STATUS_SUSPEND，system_status，可取值：AD_STATUS_NORMAL、AD_STATUS_PENDING、AD_STATUS_DENIED，adgroup_name，campaign_id。</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/targetings/get? fields=%5B%22targeting_name%22%2C%22gender%22%5D&access_token=<TOKEN>& where=%7B%22targeting_name%22%3A%2234046354897%22%7D&advertiser_id=999" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>targeting_id</td>
			<td>integer</td>
			<td>定向Id</td>
		</tr>
		<tr>
			<td>targeting_name</td>
			<td>string</td>
			<td>定向名称</td>
		</tr>
		<tr>
			<td>description</td>
			<td>string</td>
			<td>定向描述</td>
		</tr>
		<tr>
			<td>ui_visibility</td>
			<td>string</td>
			<td>定向包类型</td>
		</tr>
		<tr>
			<td>targeting_setting</td>
			<td>struct</td>
			<td>定向详细设置</td>
		</tr>
		<tr>
			<td>created_time</td>
			<td>integer</td>
			<td>创建时间</td>
		</tr>
		<tr>
			<td>last_modified_time</td>
			<td>integer</td>
			<td>最后修改时间</td>
		</tr>
		<tr>
			<td>outer_targeting_id</td>
			<td>integer</td>
			<td>外部广告定向Id</td>
		</tr>
	</tbody>
</table>
targeting_setting:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>age</td>
			<td>array</td>
			<td>年龄定向</td>
		</tr>
		<tr>
			<td>gender</td>
			<td>array</td>
			<td>性别定向（仅单选）</td>
		</tr>
		<tr>
			<td>location</td>
			<td>array</td>
			<td>商圈定向</td>
		</tr>
		<tr>
			<td>user_os</td>
			<td>array</td>
			<td>操作系统定向</td>
		</tr>
		<tr>
			<td>network_type</td>
			<td>array</td>
			<td>联网方式定向</td>
		</tr>
		<tr>
			<td>network_operator</td>
			<td>array</td>
			<td>移动运营商定向</td>
		</tr>
		<tr>
			<td>region</td>
			<td>array</td>
			<td>区域定向</td>
		</tr>
		<tr>
			<td>business_interest</td>
			<td>array</td>
			<td>商业兴趣定向</td>
		</tr>
		<tr>
			<td>online_scenario</td>
			<td>array</td>
			<td>上网场景</td>
		</tr>
		<tr>
			<td>education</td>
			<td>array</td>
			<td>用户学历</td>
		</tr>
		<tr>
			<td>paying_user_type</td>
			<td>array</td>
			<td>付费用户</td>
		</tr>
		<tr>
			<td>dressing_index</td>
			<td>array</td>
			<td>穿衣指数</td>
		</tr>
		<tr>
			<td>uv_index</td>
			<td>array</td>
			<td>紫外线指数</td>
		</tr>
		<tr>
			<td>makeup_index</td>
			<td>array</td>
			<td>化妆指数</td>
		</tr>
		<tr>
			<td>climate</td>
			<td>array</td>
			<td>气象</td>
		</tr>
		<tr>
			<td>temperature</td>
			<td>array</td>
			<td>温度（仅单温度段，且223~323）</td>
		</tr>
		<tr>
			<td>app_install_status</td>
			<td>array</td>
			<td>应用用户</td>
		</tr>
		<tr>
			<td>device_price</td>
			<td>array</td>
			<td>设备价格</td>
		</tr>
		<tr>
			<td>customized_shopping_behavior</td>
			<td>struct</td>
			<td>定制购物行为</td>
		</tr>
		<tr>
			<td>media_category_wechat</td>
			<td>array</td>
			<td>微信流量分类定向</td>
		</tr>
		<tr>
			<td>app_behavior</td>
			<td>struct</td>
			<td>app行为定向</td>
		</tr>
		<tr>
			<td>ad_placement_id</td>
			<td>array</td>
			<td>广告位(id)定向</td>
		</tr>
		<tr>
			<td>relationship_status</td>
			<td>array</td>
			<td>婚恋状态</td>
		</tr>
		<tr>
			<td>shopping_capability</td>
			<td>array</td>
			<td>消费能力</td>
		</tr>
		<tr>
			<td>customized_audience</td>
			<td>array</td>
			<td>自定义人群</td>
		</tr>
		<tr>
			<td>mobile_qq_media_follower</td>
			<td>array</td>
			<td>手Q粉丝定向</td>
		</tr>
		<tr>
			<td>keyword</td>
			<td>struct</td>
			<td>关键词定向</td>
		</tr>
		<tr>
			<td>media_category_union</td>
			<td>array</td>
			<td>移动媒体定向</td>
		</tr>
		<tr>
			<td>living_status</td>
			<td>array</td>
			<td>生活状态</td>
		</tr>
		<tr>
			<td>qzone_fans</td>
			<td>array</td>
			<td>空间粉丝定向</td>
		</tr>
		<tr>
			<td>residential_community_price</td>
			<td>array</td>
			<td>居民社区价格，单位是“元/m²”</td>
		</tr>
		<tr>
			<td>birthday_ahead_days</td>
			<td>array</td>
			<td>生日定向</td>
		</tr>
		<tr>
			<td>shopping_behavior_jd</td>
			<td>struct</td>
			<td>京东购物行为</td>
		</tr>
		<tr>
			<td>category_58</td>
			<td>array</td>
			<td>58类目定向</td>
		</tr>
		<tr>
			<td>qq_wallet_user</td>
			<td>array</td>
			<td>QQ钱包用户标签</td>
		</tr>
		<tr>
			<td>qq_wallet_shop</td>
			<td>array</td>
			<td>QQ钱包商铺标签</td>
		</tr>
		<tr>
			<td>media_category_mobile_qq</td>
			<td>array</td>
			<td>手Q兴趣部落分类定向</td>
		</tr>
		<tr>
			<td>ad_behavior</td>
			<td>array</td>
			<td>微信再营销</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":"",
			"data":{
					"list":[{
							"targeting_id":1,
							"targeting_name":"aaaaa",
							"status": "AD_STATUS_NORMAL",
							},
							...
					],
					"page_info":{
							"total_num":221,
							"total_page":11,
							"page_size":10,
							"page":1,
					}
			}
	}
										
```
##2.16 定向模块

###2.16.1 创建自定义打点LBS
************************************************
	地址：locations/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>location_type</td>
			<td>string</td>
			<td>商圈类型</td>
			<td>商圈类型</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>location_name</td>
			<td>string</td>
			<td>自定义打点名称</td>
			<td>小于等于60个英文字符，同一账户下名称不允许重复。</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>location_spec</td>
			<td>struct</td>
			<td>商圈具体配置信息</td>
			<td>商圈具体配置信息</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>city_id</td>
			<td>number</td>
			<td>整数</td>
			<td>大于0小于2^63</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "advertiser_id=999&longitude=113.971481&latitude=22.646333&radius=5000&city_id=440300&location_name=西丽" "http://sandbox.api.e.qq.com/v1.0/locations/add?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>location_id</td>
			<td>number</td>
			<td>整数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret":0,
			"msg":""，
			"data":{
				"location_id": 11000126412,
			}
	}
										
```
###2.16.2 获取自定义打点LBS列表
************************************************
	地址：locations/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>location_id</td>
			<td>integer</td>
			<td>自定义打点id</td>
			<td>自定义打点id</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
		<tr>
			<td>filter</td>
			<td>array</td>
			<td>过滤条件</td>
			<td>若此字段不传，或传空则视为无限制条件。可选过滤字段：location_name</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/locations/get?access_token=<TOKEN>&advertiser_id=999" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>location_id</td>
			<td>integer</td>
			<td>自定义打点id</td>
		</tr>
		<tr>
			<td>location_name</td>
			<td>string</td>
			<td>自定义打点名称</td>
		</tr>
		<tr>
			<td>city_id</td>
			<td>integer</td>
			<td>城市id</td>
		</tr>
		<tr>
			<td>location_spec</td>
			<td>struct</td>
			<td>商圈具体配置信息</td>
		</tr>
		<tr>
			<td>location_type</td>
			<td>string</td>
			<td>商圈类型</td>
		</tr>
	</tbody>
</table>
location_spec:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>location_type_circle</td>
			<td>struct</td>
			<td>经纬度+半径</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
		"ret":0,
		"msg":""，
		"data":{
			"list": [
				{
					"location_name": "西丽",
					"longitude": 113.971481,
					"latitude": 22.646333,
					"radius": 1500,
					"city_id": 440300
				}
				...
			],
			"page_info": {
				"page": 1,
				"page_size": 10,
				"total_num": 2,
				"total_page": 1
			}
		}
	}
										
```
###2.16.3 删除自定义打点LBS
************************************************
	地址：locations/delete
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>location_id</td>
			<td>integer</td>
			<td>自定义打点id</td>
			<td>自定义打点id</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/locations/delete?access_token=<TOKEN>" \ -d "advertiser_id=999&location_id=1100771520" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>location_id</td>
			<td>integer</td>
			<td>自定义打点id</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
		"ret":0,
		"msg":""，
		"data":{
			"location_id": 1100771520
		}
	}
										
```
##2.17 图片管理模块

###2.17.1 获取指定广告主图片记录列表
************************************************
	地址：images/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>image_id</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
			<td>32字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>filter</td>
			<td>array</td>
			<td>若此字段不传，或传空则视为无限制条件。参见：高级条件</td>
			<td>支持字段: image_signature，image_id, image_width, image_height</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/images/get?access_token=<TOKEN&advertiser_id=999" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>image_id</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
		</tr>
		<tr>
			<td>image_width</td>
			<td>integer</td>
			<td>图片宽度</td>
		</tr>
		<tr>
			<td>image_height</td>
			<td>integer</td>
			<td>图片高度</td>
		</tr>
		<tr>
			<td>image_file_size</td>
			<td>integer</td>
			<td>图片大小 单位 B(byte)</td>
		</tr>
		<tr>
			<td>image_type</td>
			<td>string</td>
			<td>图片类型</td>
		</tr>
		<tr>
			<td>image_signature</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
		</tr>
		<tr>
			<td>preview_url</td>
			<td>string</td>
			<td>预览地址</td>
		</tr>
		<tr>
			<td>outer_image_id</td>
			<td>string</td>
			<td>外部图片id</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
		"ret": 0,
		"msg": '',
		"data": {
			"list" : [{
				"advertiser_id": 999,
				"image_id" : "2644750941:4f71a5b6e71c6abf67a2b16c0b9017a8", // ID
				"image_width": 160,                                         // 宽度 pixel
				"image_height": 210,                                        // 高度 pixel
				"image_file_size": 10280,                                   // 大小 单位 B(byte)
				"image_type": "IMAGE_TYPE_JPG",                             // 类型
				"image_signature": "4f71a5b6e71c6abf67a2b16c0b9017a8",      // 签名
				"cdn_url": "http://pgdt.gtimg.cn/gdt/0/CAZW868ACBUSb-LDJoY0dbU.jpg/0?ck=4f71a5b6e71c6abf67a2b16c0b9017a8",                       // CDN地址, 只有JD调用方才返回此字段
				"preview_url": "http://pgdt.gtimg.cn/gdt/0/CAZW868ACBUSb-LDJoY0dbU.jpg/0?ck=4f71a5b6e71c6abf67a2b16c0b9017a8",                        // 预览地址
				"outer_image_id": "outer_id_xxx:v1:usable"                  // 外部图片Id
			}]，
			"page_info":{
				"total_num":221,
				"total_page":11,
				"page_size":10,
				"page":1
			}
		}
	}
										
```
###2.17.2 合并create和create_by_url接口
************************************************
	地址：images/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>from</td>
			<td>string</td>
			<td>接口类型</td>
			<td>接口类型</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>image_file</td>
			<td>file</td>
			<td>被上传的图片文件</td>
			<td>图片二进制流.支持文件类型：jpg, png, gif 文件大小限制：小于等于102400B，即100KB，单位换算规则：1KB=1024B、 1M=1024KB播放时长：gif 小于等于5s</td>
			<td>no</td>
		</tr>
		<tr>
			<td>image_signature</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
			<td>32字符</td>
			<td>no</td>
		</tr>
		<tr>
			<td>image_url</td>
			<td>string</td>
			<td>图片地址</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>outer_image_id</td>
			<td>string</td>
			<td>外部图片id</td>
			<td>1024字符内</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "advertiser_id=999&image_file=@test.jpg&image_signature=4f71a5b6e71c6abf67a2b16c0b9017a8" "http://sandbox.api.e.qq.com/v1.0/images/add?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>image_id</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
		</tr>
		<tr>
			<td>image_width</td>
			<td>number</td>
			<td>整数</td>
		</tr>
		<tr>
			<td>image_height</td>
			<td>integer</td>
			<td>图片高度</td>
		</tr>
		<tr>
			<td>image_file_size</td>
			<td>integer</td>
			<td>图片大小 单位 B(byte)</td>
		</tr>
		<tr>
			<td>image_type</td>
			<td>string</td>
			<td>图片类型</td>
		</tr>
		<tr>
			<td>image_signature</td>
			<td>string</td>
			<td>图片签名，目前使用图片的md5值</td>
		</tr>
		<tr>
			<td>outer_image_id</td>
			<td>string</td>
			<td>外部图片id</td>
		</tr>
		<tr>
			<td>preview_url</td>
			<td>string</td>
			<td>预览地址</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": '',
			"data": {
					"advertiser_id": 999,
					"image_id" : "2644750941:4f71a5b6e71c6abf67a2b16c0b9017a8",     // ID
					"image_width": 160,                                             // 宽度 pixel
					"image_height": 210,                                            // 高度 pixel
					"image_file_size": 10280,                                       // 大小 单位 B(byte)
					"image_type": "IMAGE_TYPE_JPG",                                 // 类型
					"image_signature": "4f71a5b6e71c6abf67a2b16c0b9017a8",          // 签名
					"cdn_url": "http://pgdt.gtimg.cn/gdt/0/CAZW868ACBUSb-LDJoY0dbU.jpg/0?ck=4f71a5b6e71c6abf67a2b16c0b9017a8",   // CDN地址, 只有JD调用方才返回此字段
					"preview_url": "http://pgdt.gtimg.cn/gdt/0/CAZW868ACBUSb-LDJoY0dbU.jpg/0?ck=4f71a5b6e71c6abf67a2b16c0b9017a8",   // 预览地址
					"outer_image_id": "outer_id_xxx:v1:usable"                      // 外部图片Id
			}
	}
										
```
##2.18 媒体管理模块

###2.18.1 创建媒体
************************************************
	地址：videos/add
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>media_file</td>
			<td>file</td>
			<td>被上传的媒体文件</td>
			<td>媒体二进制流</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>media_signature</td>
			<td>string</td>
			<td>媒体签名，目前使用媒体的md5值</td>
			<td>32字符</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>media_description</td>
			<td>string</td>
			<td>流媒体描述</td>
			<td>小于255字符</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "advertiser_id=999&media_file=@test.jpg&media_signature=4f71a5b6e71c6abf67a2b16c0b9017a8" "http://sandbox.api.e.qq.com/v1.0/videos/add?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>media_id</td>
			<td>integer</td>
			<td>媒体Id</td>
		</tr>
		<tr>
			<td>repeat</td>
			<td>integer</td>
			<td>是否为重复上传</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": '',
			"data": {
					"media_id" : 15946,     // ID
					"repeat": 0,    //新媒体，未重复
			 }
	}
										
```
###2.18.2 广告主预览媒体文件
************************************************
	地址：videos/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>media_id</td>
			<td>integer</td>
			<td>媒体Id</td>
			<td>小于2^63</td>
			<td>yes</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/videos/get?access_token=<TOKEN>&advertiser_id=999&media_id=91312" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>media_id</td>
			<td>integer</td>
			<td>媒体Id</td>
		</tr>
		<tr>
			<td>media_width</td>
			<td>integer</td>
			<td>媒体宽度</td>
		</tr>
		<tr>
			<td>media_height</td>
			<td>integer</td>
			<td>媒体高度</td>
		</tr>
		<tr>
			<td>video_frames</td>
			<td>integer</td>
			<td>视频帧数</td>
		</tr>
		<tr>
			<td>video_fps</td>
			<td>integer</td>
			<td>视频帧率</td>
		</tr>
		<tr>
			<td>video_codec</td>
			<td>string</td>
			<td>视频格式</td>
		</tr>
		<tr>
			<td>video_bit_rate</td>
			<td>integer</td>
			<td>视频码率</td>
		</tr>
		<tr>
			<td>audio_codec</td>
			<td>string</td>
			<td>音频格式</td>
		</tr>
		<tr>
			<td>audio_bit_rate</td>
			<td>integer</td>
			<td>音频码率</td>
		</tr>
		<tr>
			<td>media_file_size</td>
			<td>integer</td>
			<td>媒体文件大小 单位 B(byte)</td>
		</tr>
		<tr>
			<td>media_type</td>
			<td>string</td>
			<td>媒体类型</td>
		</tr>
		<tr>
			<td>media_signature</td>
			<td>string</td>
			<td>媒体签名，目前使用媒体的md5值</td>
		</tr>
		<tr>
			<td>system_status</td>
			<td>string</td>
			<td>转码状态</td>
		</tr>
		<tr>
			<td>media_description</td>
			<td>string</td>
			<td>流媒体描述</td>
		</tr>
		<tr>
			<td>preview_url</td>
			<td>string</td>
			<td>流媒体预览地址</td>
		</tr>
	</tbody>
</table>
返回示例：
```
file data
```
##2.19 定向标签模块

定向标签模块描述

###2.19.1 合并获得targeting标签列表的接口
描述信息
************************************************
	地址：targeting_tags/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>conf</td>
			<td>string</td>
			<td>接口类型</td>
			<td>接口类型</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>region_id</td>
			<td>integer</td>
			<td>城市ID</td>
			<td>城市ID是六位的数字</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/targeting_tags/get?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>integer</td>
			<td>商圈</td>
		</tr>
		<tr>
			<td>name</td>
			<td>string</td>
			<td>商圈名称</td>
		</tr>
		<tr>
			<td>parent_id</td>
			<td>integer</td>
			<td>父节点ID</td>
		</tr>
		<tr>
			<td>lbs_type</td>
			<td>string</td>
			<td>LBS类型</td>
		</tr>
		<tr>
			<td>available_for_individual</td>
			<td>boolean</td>
			<td>行业是否可用于个人客户注册</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
		"ret" : 0,                   // 返回码 0为OK
		"msg" : "",                  // 返回说明
		"data" : {
			"list" : [
				 {
					"id" : "110000",
					"name":"北京市",
				 },
				 {
					"id" : "120000",
					"name":"天津市",
				 },
				......
			]
		}
	}
										
```
##2.20 工具类模块

###2.20.1 覆盖人数预估
************************************************
	地址：estimation/get
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>targeting_setting</td>
			<td>struct</td>
			<td>定向详细设置</td>
			<td>存放所有定向条件</td>
			<td>no</td>
		</tr>
		<tr>
			<td>adgroup_setting</td>
			<td>struct</td>
			<td>广告组信息所组成的对象</td>
			<td>小于1024英文字符，支持字段time_series, site_set, bid_type, bid, product_refs_id, product_type，示例：{"bid_type":"COSTTYPE_CPC", "product_type": "PRODUCT_TYPE_LINK"}</td>
			<td>no</td>
		</tr>
		<tr>
			<td>creative_setting</td>
			<td>array</td>
			<td>素材信息所组成的对象</td>
			<td>小于1024英文字符，支持字段creative_template_id，[{"creative_template_id":1},{"creative_template_id":2}]</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
	注：至少包含一个预估条件

请求示例:
```
 curl -d "targeting_spec=%7B%22region%22%3A%5B710000%2C510300%5D%7D&advertiser_id=999" "http://sandbox.api.e.qq.com/v1.0/estimation/get?access_token=<TOKEN>" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>approximate_count</td>
			<td>integer</td>
			<td>预估人数</td>
		</tr>
		<tr>
			<td>impression</td>
			<td>integer</td>
			<td>预估日曝光量</td>
		</tr>
		<tr>
			<td>min_bid_amount</td>
			<td>integer</td>
			<td>建议出价最小值</td>
		</tr>
		<tr>
			<td>max_bid_amount</td>
			<td>integer</td>
			<td>建议出价最大值</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": '',
			"data": {
					"approximate_count": 1023288, // 预估人数
					"impression": 102311, // 预估日曝光量
					"min_bid_amount":117,// 最低出价建议
					"max_bid_amount":199, // 最高出价建议
			}
	}
										
```
##2.21 广告素材模块

###2.21.1 获取规格描述
************************************************
	地址：creative_templates/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>creative_template_id</td>
			<td>integer</td>
			<td>素材规格Id</td>
			<td>详见 [creative_template_id | 素材规格Id]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>site_set</td>
			<td>array</td>
			<td>投放站点集合</td>
			<td>当前仅支持单站点，取值详见 [site_set_definition | 投放站点集合]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>product_type</td>
			<td>string</td>
			<td>标的物类型</td>
			<td>详见 [product_type | 标的物类型]</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -d "creative_template_id=176" "http://sandbox.api.e.qq.com/v1.0/creative_templates/get?access_token=&lt;TOKEN&gt;" 
```
返回示例：
```

	{
		"ret": 0,
		"msg": "",
		"data": {
			"list": [
				{
					"creative_template_id": 176,
					"creative_preview": [],
					"creative_template": {
						"title": {
							"default": "",
							"fixed": "",
							"name": "title",
							"desc": "标题",
							"comment": "",
							"use": "required",
							"min_occurs": 1,
							"max_occurs": 1,
							"type": "TEXT",
							"dimension": {
								"content": {
									"min_length": "0",
									"max_length": "6"
								}
							}
						},
						"description": {
							"default": "",
							"fixed": "",
							"name": "description",
							"desc": "文案",
							"comment": "",
							"use": "required",
							"min_occurs": 1,
							"max_occurs": 1,
							"type": "TEXT",
							"dimension": {
								"content": {
									"min_length": "0",
									"max_length": "20"
								}
							}
						},
						"top_icon": {
							"default": "",
							"fixed": "",
							"name": "top_icon",
							"desc": "顶部图标",
							"comment": "",
							"use": "required",
							"min_occurs": 1,
							"max_occurs": 1,
							"type": "IMAGE",
							"dimension": {
								"resource": {
									"width": "30",
									"height": "30",
									"file_size_KB_limit": "1024",
									"file_format": "*.jpg|*.jpeg|*.png",
									"min_duration": "",
									"max_duration": "",
									"duration_type": ""
								}
							}
						},
						"image": {
							"default": "",
							"fixed": "",
							"name": "image",
							"desc": "图片",
							"comment": "",
							"use": "required",
							"min_occurs": 1,
							"max_occurs": 1,
							"type": "IMAGE",
							"dimension": {
								"resource": {
									"width": "1000",
									"height": "400",
									"file_size_KB_limit": "1024",
									"file_format": "*.jpg|*.jpeg|*.png",
									"min_duration": "",
									"max_duration": "",
									"duration_type": ""
								}
							}
						},
						"button_text": {
							"default": "",
							"fixed": "",
							"name": "button_text",
							"desc": "按钮文字",
							"comment": "",
							"use": "required",
							"min_occurs": 1,
							"max_occurs": 1,
							"type": "TEXT",
							"dimension": {
								"content": {
									"min_length": "2",
									"max_length": "4"
								}
							}
						},
						"bottom_text": {
							"default": "",
							"fixed": "",
							"name": "bottom_text",
							"desc": "底部文字",
							"comment": "",
							"use": "required",
							"min_occurs": 1,
							"max_occurs": 1,
							"type": "TEXT",
							"dimension": {
								"content": {
									"min_length": "6",
									"max_length": "8"
								}
							}
						},
						"activity_type": {
							"type": "ENUM",
							"name": "activity_type",
							"default": "",
							"fixed": "",
							"use": "required",
							"min_occurs": 1,
							"max_occurs": 1,
							"dimension": {
								"option": [
									"BIRTHDAY_ACTIVITY_PAGE_CARD",
									"FESTIVAL_ACTIVITY_PAGE_CARD"
								]
							},
							"desc": {
								"BIRTHDAY_ACTIVITY_PAGE_CARD": "生日页卡",
								"FESTIVAL_ACTIVITY_PAGE_CARD": "节日页卡"
							},
							"comment": {
								"BIRTHDAY_ACTIVITY_PAGE_CARD": "亲爱的XXX（昵称），你还有xx天就生日啦！",
								"FESTIVAL_ACTIVITY_PAGE_CARD": "亲爱的XXX（昵称），"
							}
						}
					}
				}
			]
		}
	}
										
```
##2.22 通用人群管理

###2.22.1 人群查询
************************************************
	地址：custom_audiences/get
	方式：POST
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>account_id</td>
			<td>integer</td>
			<td>账户ID</td>
			<td></td>
			<td>yes</td>
		</tr>
		<tr>
			<td>query_parameter</td>
			<td>struct</td>
			<td>查询参数</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_parameter</td>
			<td>struct</td>
			<td>分页参数</td>
			<td></td>
			<td>no</td>
		</tr>
		<tr>
			<td>orderby_field</td>
			<td>string</td>
			<td>排序字段</td>
			<td>只支持ID,NAME</td>
			<td>no</td>
		</tr>
		<tr>
			<td>orderby_desc</td>
			<td>boolean</td>
			<td>是否降序</td>
			<td></td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl -X POST -H "Authorization: Bearer <YOUR TOKEN>" -H "Content-Type: application/json;charset=utf8" -d '{ "account_id": 25610, "query_parameter" : { "audience_id":[20243,20699,21304], "targeting_enable_status" : "ENABLED" } }' "http://sandbox.api.e.qq.com/v1.0/custom_audiences/get" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>total</td>
			<td>integer</td>
			<td>记录总数</td>
		</tr>
		<tr>
			<td>data_list</td>
			<td>array</td>
			<td>人群列表</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
		"code": 0,
		"message": "ok",
		"data": {
			"total": 3,
			"data_list": [
				{
					"audience_id": 20243,
					"name": "read接口Demo1",
					"description": "",
					"type": "COMBINE",
					"targeting_enabled": true,
					"process_status": "VALID",
					"time_window": "LAST_YEAR",
					"user_type": "COMBINE",
					"created_time": "2016-05-16 16:22:39",
					"user_number": 10541033,
					"is_own": true,
					"data_source_id": 0,
					"appendable": false,
					"lookalike_number": 0,
					"magnification": 0
				},
				{
					"audience_id": 20699,
					"name": "read接口Demo2",
					"description": "",
					"type": "PACKAGE",
					"targeting_enabled": true,
					"process_status": "VALID",
					"time_window": "LAST_YEAR",
					"user_type": "PACKAGE",
					"id_type": [
					  "QQ"
					],
					"created_time": "2016-05-19 14:41:37",
					"user_number": 10218284,
					"is_own": true,
					"appendable": true,
					"lookalike_number": 0,
					"magnification": 0
				},
				{
					"audience_id": 21304,
					"name": "052401",
					"description": "通过lookalike进行了相似人群扩展",
					"type": "LOOKALIKE",
					"targeting_enabled": true,
					"process_status": "VALID",
					"time_window": "LAST_YEAR",
					"user_type": "LOOKALIKE",
					"created_time": "2016-05-24 14:59:00",
					"user_number": 6145766,
					"is_own": true,
					"appendable": false,
					"lookalike_number": 6090000,
					"magnification": 0
				}
			]
		}
	}
										
```
##2.23 通用人群管理

##2.24 报表模块

###2.24.1 合并所有daily report接口
************************************************
	地址：daily_reports/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>level</td>
			<td>string</td>
			<td>接口类型</td>
			<td>接口类型</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>date_range</td>
			<td>struct</td>
			<td>时间范围</td>
			<td>日期格式，{"start_date":"2014-03-01","end_date":"2014-04-02"}</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
		<tr>
			<td>group_by</td>
			<td>array</td>
			<td>聚合参数，例：["date"]</td>
			<td>见 [group_by | 聚合规则定义]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>adgroup_id_list</td>
			<td>array</td>
			<td>如[2001,2002,2003,2004]，可不填</td>
			<td>数量不能不超过200个</td>
			<td>no</td>
		</tr>
		<tr>
			<td>campaign_id_list</td>
			<td>array</td>
			<td>如[2001,2002,2003,2004]，可不填</td>
			<td>数量不能不超过200个</td>
			<td>no</td>
		</tr>
		<tr>
			<td>creative_id_list</td>
			<td>array</td>
			<td>如[2001,2002,2003,2004]，可不填</td>
			<td>数量不能不超过200个</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/daily_reports/get?access_token=<TOKEN>& advertiser_id=999&date_range=%7B%22start_date%22%3A%222014-04-11%22%2C%22end_date%22%3A%222014-04-15%22%7D" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>date</td>
			<td>string</td>
			<td>查询时间</td>
		</tr>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
		</tr>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
		</tr>
		<tr>
			<td>impression</td>
			<td>integer</td>
			<td>曝光量</td>
		</tr>
		<tr>
			<td>click</td>
			<td>integer</td>
			<td>点击量</td>
		</tr>
		<tr>
			<td>ctr</td>
			<td>float</td>
			<td>点击率</td>
		</tr>
		<tr>
			<td>cpc</td>
			<td>float</td>
			<td>点击均价</td>
		</tr>
		<tr>
			<td>cpm</td>
			<td>float</td>
			<td>每千次曝光消耗</td>
		</tr>
		<tr>
			<td>cost</td>
			<td>integer</td>
			<td>消耗</td>
		</tr>
		<tr>
			<td>download</td>
			<td>integer</td>
			<td>APP下载量</td>
		</tr>
		<tr>
			<td>conversion</td>
			<td>integer</td>
			<td>转化量（APP安装量）</td>
		</tr>
		<tr>
			<td>activation</td>
			<td>integer</td>
			<td>APP激活量</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
					"list":[{
							"date":"2014-03-14",
							"impression": 12000,//曝光
							"click": 1200,//点击
							"ctr": 0.00301,//CTR CTR=（点击/曝光），例如这里CTR返回为0.00301，界面展示应该为 0.301%
							"cpc":20,//点击均价,单位为分
							"cpm":200,// CPM=消耗/(曝光/1000),每千次曝光消耗，单位为分
							"cost": 1200,//消耗，单位为分
							},
							{...}
					],
					"page_info":{
							"total_num":221,// 总条数
							"total_page":11,// 总页数
							"page_size":20,// 每页显示条数
							"page":1,// 当前页码
					}
			}
	}
										
```
##2.25 报表模块

###2.25.1 合并所有hourly的报表
************************************************
	地址：hourly_reports/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>level</td>
			<td>string</td>
			<td>接口类型</td>
			<td>接口类型</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>date</td>
			<td>string</td>
			<td>查询时间</td>
			<td>日期格式，如2014-03-01</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
		<tr>
			<td>group_by</td>
			<td>array</td>
			<td>聚合参数，例：["time"]</td>
			<td>见 [group_by | 聚合规则定义]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>campaign_id_list</td>
			<td>array</td>
			<td>如[2001,2002,2003,2004]，可不填</td>
			<td>数量不能不超过200个</td>
			<td>no</td>
		</tr>
		<tr>
			<td>adgroup_id_list</td>
			<td>array</td>
			<td>如[2001,2002,2003,2004]，可不填</td>
			<td>数量不能不超过200个</td>
			<td>no</td>
		</tr>
		<tr>
			<td>creative_id_list</td>
			<td>array</td>
			<td>如[2001,2002,2003,2004]，可不填</td>
			<td>数量不能不超过200个</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/hourly_reports/get?access_token=<TOKEN>& advertiser_id=999&date=2014-04-10" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>hour</td>
			<td>integer</td>
			<td>小时(0-23)</td>
		</tr>
		<tr>
			<td>impression</td>
			<td>integer</td>
			<td>曝光量</td>
		</tr>
		<tr>
			<td>campaign_id</td>
			<td>integer</td>
			<td>推广计划Id</td>
		</tr>
		<tr>
			<td>adgroup_id</td>
			<td>integer</td>
			<td>广告组Id</td>
		</tr>
		<tr>
			<td>creative_id</td>
			<td>integer</td>
			<td>广告素材Id</td>
		</tr>
		<tr>
			<td>click</td>
			<td>integer</td>
			<td>点击量</td>
		</tr>
		<tr>
			<td>ctr</td>
			<td>float</td>
			<td>点击率</td>
		</tr>
		<tr>
			<td>cpc</td>
			<td>float</td>
			<td>点击均价</td>
		</tr>
		<tr>
			<td>cpm</td>
			<td>float</td>
			<td>每千次曝光消耗</td>
		</tr>
		<tr>
			<td>cost</td>
			<td>integer</td>
			<td>消耗</td>
		</tr>
		<tr>
			<td>download</td>
			<td>integer</td>
			<td>APP下载量</td>
		</tr>
		<tr>
			<td>conversion</td>
			<td>integer</td>
			<td>转化量（APP安装量）</td>
		</tr>
		<tr>
			<td>activation</td>
			<td>integer</td>
			<td>APP激活量</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
					"list": [{
							"hour":"00:00",
							"impression": 12000,//曝光
							"click": 1200,//点击
							"ctr": 0.00301,//CTR CTR=（点击/曝光），例如这里CTR返回为0.00301，界面展示应该为 0.301%
							"cpc":20,//点击均价,单位为分
							"cpm":200,// CPM=消耗/(曝光/1000),每千次曝光消耗，单位为分
							"cost": 1200,//消耗，单位为分
					},
					{
							"hour": "2",
							"impression": 12000,//曝光
							"click": 1200,//点击
							"ctr": 0.00301,//CTR CTR=（点击/曝光），例如这里CTR返回为0.00301，界面展示应该为 0.301%
							"cpc":20,//点击均价,单位为分
							"cpm":200,// CPM=消耗/(曝光/1000),每千次曝光消耗，单位为分
							"cost": 1200,//消耗，单位为分
					}
					....
			],
					"page_info":{
							"total_num":24,// 总条数
							"total_page":2,// 总页数
							"page_size":10,// 每页显示条数
							"page":1,// 当前页码
					}
			}
	}
										
```
##2.26 报表模块

###2.26.1 合并gender,age,region报表
************************************************
	地址：targeting_tag_reports/get
	方式：GET
参数：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
			<th>限制</th>
			<th>是否必填</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>tag</td>
			<td>string</td>
			<td>接口类型</td>
			<td>接口类型</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>advertiser_id</td>
			<td>integer</td>
			<td>广告主ID</td>
			<td>详见附录</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>date_range</td>
			<td>struct</td>
			<td>时间范围</td>
			<td>日期格式，{"start_date":"2014-03-01","end_date":"2014-04-02"}</td>
			<td>yes</td>
		</tr>
		<tr>
			<td>group_by</td>
			<td>array</td>
			<td>聚合参数，例：["date"]</td>
			<td>见 [group_by | 聚合规则定义]</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>搜索页码</td>
			<td>大于等于1，若不传则视为1</td>
			<td>no</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
			<td>大于等于1，且小于100，若不传则视为10</td>
			<td>no</td>
		</tr>
		<tr>
			<td>campaign_id_list</td>
			<td>array</td>
			<td>如[2001,2002,2003,2004]，可不填</td>
			<td>数量不能不超过200个</td>
			<td>no</td>
		</tr>
		<tr>
			<td>adgroup_id_list</td>
			<td>array</td>
			<td>如[2001,2002,2003,2004]，可不填</td>
			<td>数量不能不超过200个</td>
			<td>no</td>
		</tr>
	</tbody>
</table>
请求示例:
```
 curl "http://sandbox.api.e.qq.com/v1.0/targeting_tag_reports/get?access_token=<TOKE>& advertiser_id=999&date_range=%7B%22start_date%22%3A%222014-03-02%22%2C%22end_date%22%3A%222014-03-31%22%7D" 
```
返回字段：
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>list</td>
			<td>array</td>
			<td>列表信息</td>
		</tr>
		<tr>
			<td>page_info</td>
			<td>struct</td>
			<td>分页配置信息</td>
		</tr>
	</tbody>
</table>
list:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>date</td>
			<td>string</td>
			<td>查询时间</td>
		</tr>
		<tr>
			<td>impression</td>
			<td>integer</td>
			<td>曝光量</td>
		</tr>
		<tr>
			<td>click</td>
			<td>integer</td>
			<td>点击量</td>
		</tr>
		<tr>
			<td>ctr</td>
			<td>float</td>
			<td>点击率</td>
		</tr>
		<tr>
			<td>cpc</td>
			<td>float</td>
			<td>点击均价</td>
		</tr>
		<tr>
			<td>cpm</td>
			<td>float</td>
			<td>每千次曝光消耗</td>
		</tr>
		<tr>
			<td>cost</td>
			<td>integer</td>
			<td>消耗</td>
		</tr>
		<tr>
			<td>download</td>
			<td>integer</td>
			<td>APP下载量</td>
		</tr>
		<tr>
			<td>conversion</td>
			<td>integer</td>
			<td>转化量（APP安装量）</td>
		</tr>
		<tr>
			<td>activation</td>
			<td>integer</td>
			<td>APP激活量</td>
		</tr>
		<tr>
			<td>gender</td>
			<td>string</td>
			<td>性别</td>
		</tr>
		<tr>
			<td>age</td>
			<td>string</td>
			<td>年龄段中文</td>
		</tr>
		<tr>
			<td>region_id</td>
			<td>integer</td>
			<td>省份ID</td>
		</tr>
		<tr>
			<td>province</td>
			<td>string</td>
			<td>省份中文</td>
		</tr>
	</tbody>
</table>
page_info:
<table style="border-collapse: collapse; border: 1px solid #efg; width: 100%">
	<thead>
		<tr>
			<th>名称</th>
			<th>类型</th>
			<th>描述</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>page</td>
			<td>integer</td>
			<td>页码</td>
		</tr>
		<tr>
			<td>page_size</td>
			<td>integer</td>
			<td>一页显示的数据条数</td>
		</tr>
		<tr>
			<td>total_num</td>
			<td>integer</td>
			<td>总条数</td>
		</tr>
		<tr>
			<td>total_page</td>
			<td>integer</td>
			<td>总页数</td>
		</tr>
	</tbody>
</table>
返回示例：
```

	{
			"ret": 0,
			"msg": "",
			"data": {
					"list":[{
							"date":"2014-03-14",
							"impression": 12000,//曝光
							"click": 1200,//点击
							"ctr": 0.00301,//CTR CTR=（点击/曝光），例如这里CTR返回为0.00301，界面展示应该为 0.301%
							"cpc":20,//点击均价,单位为分
							"cpm":200,// CPM=消耗/(曝光/1000),每千次曝光消耗，单位为分
							"cost": 1200,//消耗，单位为分
							"gender" : "FEMALE", // 性别
					},
					{...}

					],
					"page_info":{
							"total_num":24,// 总条数
							"total_page":2,// 总页数
							"page_size":20,// 每页显示条数
							"page":1,// 当前页码
					}
			}
	}
										
```
#三、附录

<div id='speed_mode'></div>
###标准投放类型 | speed_mode
********************************************



	SPEEDMODE_FAST      加速播放
	SPEEDMODE_STANDARD  标准播放
						

<div id='bid_type'></div>
###扣费方式 | bid_type
********************************************



	COSTTYPE_CPC     CPC扣费
	COSTTYPE_CPA     CPA扣费
	COSTTYPE_CPS     CPS扣费
	COSTTYPE_CPM     CPM扣费
	COSTTYPE_CPD     CPD扣费
						

<div id='product_type'></div>
###标的物类型 | product_type
********************************************



	PRODUCT_TYPE_LINK   腾讯域以外的连接——网站链接
						

<div id='creative_selection_type'></div>
###素材播放模式 | creative_selection_type
********************************************



	PLAYMODE_BY_TURNS			轮询播放
	PLAYMODE_AUTO_OPTIMIZED		自动优化播放
						

<div id='creative_combination_type'></div>
###广告类型 | creative_combination_type
********************************************



	ADGROUP_CREATIVE_COMBINATION_NORMAL   普通广告
				ADGROUP_CREATIVE_COMBINATION_CONTAINTER    集装箱广告
				COMBINATION_TYPE_DYNAMIC    动态创意广告

<div id='creative_template_id'></div>
###素材规格Id | creative_template_id
********************************************



	注：括号内的长度是指字符意义上的长度上限，如“展示A广告”，长度为5
	
	creative_template_id: 1     宽高：140/226  必填项:image_url
	creative_template_id: 2     宽高：160/210  必填项:title(18)，image_url
	creative_template_id: 5     宽高：125/90   必填项:title(9)，description(30)，image_url
	creative_template_id: 8     宽高：140/140  必填项:image_url
	creative_template_id: 9     宽高：300/80   必填项:image_url
	creative_template_id: 10    宽高：640/100  必填项:image_url
	creative_template_id: 11    宽高：728/90   必填项:image_url
	creative_template_id: 12    宽高：198/40   必填项:image_url
	creative_template_id: 13    宽高：760/75   必填项:image_url
	creative_template_id: 14    宽高：565/250  必填项:image_url
	creative_template_id: 19    宽高：230/92   必填项:image_url
	creative_template_id: 20    宽高：960/75   必填项:image_url
	creative_template_id: 21    宽高：220/120  必填项:image_url
	creative_template_id: 22    宽高：140/40   必填项:image_url
	creative_template_id: 23    宽高：198/100  必填项:title(18)，image_url
	creative_template_id: 24    宽高：75/75    必填项:title(10)，description(24)，image_url
	creative_template_id: 25    宽高：0/0  必填项:title(14)，description(30)
	creative_template_id: 27    宽高：160/120  必填项:title(13)，image_url
	creative_template_id: 28    宽高：240/38   必填项:image_url
	creative_template_id: 29    宽高：0/0  必填项:title(14)
	creative_template_id: 30    宽高：336/280  必填项:image_url
	creative_template_id: 31    宽高：480/75   必填项:image_url
	creative_template_id: 32    宽高：300/250  必填项:image_url
	creative_template_id: 33    宽高：75/75    必填项:image_url
	creative_template_id: 34    宽高：264/54   必填项:image_url
	creative_template_id: 35    宽高：320/50   必填项:image_url
	creative_template_id: 37    宽高：142/185  必填项:image_url
	creative_template_id: 39    宽高：120/600  必填项:image_url
	creative_template_id: 42    宽高：140/40   必填项:description(30)，image_url
	creative_template_id: 44    宽高：0/0  必填项:title(30)，description(30)
	creative_template_id: 46    宽高：0/0  必填项:title(16)
	creative_template_id: 47    宽高：0/0  必填项:title(14)
	creative_template_id: 48    宽高：75/75    必填项:title(18)，image_url
	creative_template_id: 50    宽高：72/72    必填项:title(12)，description(15)，image_url
	creative_template_id: 51    宽高：0/0  必填项:title(20)，description(30)
	creative_template_id: 52    宽高：320/50   必填项:image_url
	creative_template_id: 53    宽高：640/100  必填项:image_url
	creative_template_id: 54    宽高：200/200  必填项:image_url
	creative_template_id: 56    宽高：0/0  必填项:title(20)
	creative_template_id: 58    宽高：300/250  必填项:image_url
	creative_template_id: 59    宽高：600/500  必填项:image_url
	creative_template_id: 60    宽高：135/100  必填项:image_url
	creative_template_id: 61    宽高：970/90   必填项:image_url
	creative_template_id: 62    宽高：468/60   必填项:image_url
	creative_template_id: 64    宽高：180/90   必填项:image_url
	creative_template_id: 65    宽高：1000/560 必填项:title(30)，image_url
	creative_template_id: 66    宽高：512/120  必填项:title(30)，image_url
	creative_template_id: 67    宽高：80/60    必填项:title(18)，image_url
	creative_template_id: 68    宽高：0/0  必填项:title(13)，description(26)
	creative_template_id: 69    宽高：72/72    必填项:title(14)，description(14)，image_url
	creative_template_id: 70    宽高：72/72    必填项:title(12)，description(80)，image_url
	creative_template_id: 71    宽高：0/0  必填项:title(15)，description(15)
	creative_template_id: 72    宽高：160/110  必填项:title(13)，image_url
	creative_template_id: 73    宽高：370/70   必填项:title(13)，image_url
	creative_template_id: 74    宽高：0/0  必填项:title(13)
	creative_template_id: 75    宽高：50/536   必填项:image_url
	creative_template_id: 78    宽高：0/0  必填项:
	creative_template_id: 79    宽高：640/960  必填项:image_url
	creative_template_id: 80    宽高：320/480  必填项:image_url
	creative_template_id: 81    宽高：0/0  必填项:
	creative_template_id: 82    宽高：220/120  必填项:image_url
	creative_template_id: 83    宽高：0/0  必填项:title(13)
	creative_template_id: 84    宽高：320/180  必填项:title(13)，image_url
	creative_template_id: 85    宽高：320/152  必填项:title(13)，image_url
	creative_template_id: 86    宽高：960/90   必填项:image_url
	creative_template_id: 87    宽高：0/0  必填项:
	creative_template_id: 88    宽高：80/80    必填项:title(30)，image_url
	creative_template_id: 89    宽高：120/120  必填项:title(14)，description(14)，image_url
	creative_template_id: 90    宽高：120/120  必填项:title(12)，description(80)，image_url
	creative_template_id: 91    宽高：0/0  必填项:title(15)
	creative_template_id: 94    宽高：50/50    必填项:title(22)，image_url
	creative_template_id: 95    宽高：210/100  必填项:image_url
	creative_template_id: 96    宽高：140/425  必填项:image_url
	creative_template_id: 99    宽高：200/162  必填项:image_url
	creative_template_id: 100   宽高：200/258  必填项:image_url
	creative_template_id: 101   宽高：400/100  必填项:image_url
	creative_template_id: 102   宽高：130/60   必填项:image_url
	creative_template_id: 103   宽高：69/160   必填项:image_url
	creative_template_id: 104   宽高：660/70   必填项:image_url
	creative_template_id: 105   宽高：310/68   必填项:image_url
	creative_template_id: 106   宽高：690/270  必填项:image_url
	creative_template_id: 107   宽高：395/70   必填项:image_url
	creative_template_id: 108   宽高：465/230  必填项:image_url
	creative_template_id: 109   宽高：210/100  必填项:title(14)，image_url
	creative_template_id: 133   宽高：582/166  必填项:image_url
	creative_template_id: 134   宽高：144/144  必填项:title(14)，description（28），image_url
						

<div id='group_by'></div>
###聚合规则定义 | group_by
********************************************



	天报表支持的聚合条件，默认按date聚合
	date
	adgroup_id
	
	小时报表支持的聚合条件，默认按time聚合
	time
	adgroup_id
	
	在报表接口中，聚合支持如下规则：
	（1）广告主维度天报表，默认按天聚合（GROUP BY date）
	（2）广告主小时维度报表，默认按小时聚合 （GROUP BY time）
	（3）广告维度天报表，支持按天、按广告聚合，默认按天聚合 （GROUP BY date）
			应用场景举例：
					*如选择了按天聚合，有11个广告，日期有10天，则一共有10条记录，返回数据节点中的adgroup_id无意义
					*如选择了按广告聚合（（GROUP BY adgroup_id）），有11个广告，日期有10天，则只有 11 条数据，各项指标为10天综合指标，如曝光为10天的曝光加和，返回的数据节点中的date（天）无意义
					*如需查看某个广告 10天的趋势图，则 adgroup_id_list 传该广告ID，并 group by date 即可
	
	（4）广告维度小时报表，支持按小时、按广告聚合
					*如选择了按小时聚合，有11个广告，当天有24小时,则一共有24条数据,返回数据节点中的adgroup_id无意义
					*如选择了按广告聚合（（GROUP BY adgroup_id）），有11个广告，则只有 11 条数据，各项指标为当天综合指标，如曝光为24小时的曝光加和，返回的数据节点中的hour（小时）无意义
					*如需查看某个广告 某天的趋势图，则 adgroup_id_list 传该广告ID，并 group by time 即可
						

