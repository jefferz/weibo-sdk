# Overview #
This project is used for the Sina weibo's third-party application network request library,Please read follow document before you start use it.<br>
<a href='http://open.weibo.com/wiki/API%E6%96%87%E6%A1%A3_V2'>http://open.weibo.com/wiki/API%E6%96%87%E6%A1%A3_V2</a><br>

<h1>Requirement</h1>

<h2>Default IDE support</h2>
1. WIN32(VS2008)<br>
2. Mac OS or iOS(XCode)<br>
3. Unix/Linux(Automake)<br>

<h2>Require library</h2>
1. Boost<br>
2. LibCurl<br>
3. SSL<br>
4. CPPUnit (For UnitTest, if don't need Unittest, please ignore) <br>
<br>
<br>

<h1>Using weibo SDK</h1>
<br>
<pre><code>
#include &lt;weibo/IWeibo.hxx&gt;
#include &lt;weibo/IWeiboMethod.hxx&gt;

using namespace weibo;

boost::shared_ptr&lt;weibo::IWeibo&gt; weiboPtr;
static bool logined = false;

void OnDelegateComplated(unsigned int methodOption, const char* httpHeader,
						 ParsingObject* result, const UserTaskInfo* pTask);
void OnDelegateErrored(unsigned int methodOption, const int errCode, const int subErrCode, 
					   ParsingObject* result, const UserTaskInfo* pTask);
void OnDelegateWillRelease(unsigned int methodOption, const UserTaskInfo* pTask);

void main()
{
	weiboPtr = weibo::WeiboFactory::getWeibo();
	weiboPtr-&gt;startup();
	
	weibo::IWeiboMethod* method = weiboPtr-&gt;getMethod();

	// Please assin your consumer key and consumer secret
	weiboPtr-&gt;setOption(weibo::WOPT_CONSUMER, "YOUR CONSUMER KEY", "YOUR CONSUMER SECRET");

	// Ansy request login, see OnDelegateComplated
	method-&gt;oauth2("userName", "password", NULL);


	// Delegate option
	weiboPtr-&gt;OnDelegateComplated = &amp;OnDelegateComplated;
	weiboPtr-&gt;OnDelegateErrored = &amp;OnDelegateErrored;
	//weiboPtr-&gt;OnDelegateWillRelease = OnDelegateWillRelease;

	// Must waiting for a moment.
	Sleep(5000);

	// Go! Send weibo, Especially, text  MUST convert to UTF8 when it is UNICODE character set.
	method-&gt;postStatusesUpdate("This is a weibo.....", NULL, NULL);

	// Must waiting for a moment.
	Sleep(5000);

	weiboPtr-&gt;stopAll();
	weiboPtr-&gt;shutdown();

	return 0;
};


void OnDelegateComplated(unsigned int methodOption, const char* httpHeader,
						 ParsingObject* result, const UserTaskInfo* pTask)
{
	if (methodOption == WBOPT_OAUTH2_ACCESS_TOKEN) 
	{
		if (result-&gt;isUseable())
		{
			logined = true;
			std::string access_token = result-&gt;getSubStringByKey("access_token");

			// Note: Must set acess token to sdk!
			weiboPtr-&gt;setOption(WOPT_ACCESS_TOKEN, access_token.c_str());
		}
	}
	else if (methodOption == WBOPT_POST_STATUSES_UPDATE)
	{
		// Send weibo successed!
		// ...
	}
}

void OnDelegateErrored(unsigned int methodOption, const int errCode, const int subErrCode, 
	  ParsingObject* result, const UserTaskInfo* pTask)
{
	// Please reference http://open.weibo.com/wiki/Help/error
	if (methodOption == WBOPT_OAUTH2_ACCESS_TOKEN) 
	{
		if (result &amp;&amp; result-&gt;isUseable())
		{
			std::string error_code = result-&gt;getSubStringByKey("error_code");
			std::string request = result-&gt;getSubStringByKey("request");
			std::string error = result-&gt;getSubStringByKey("error");
		}
	}
	else if (methodOption == WBOPT_POST_STATUSES_UPDATE)
	{
		// Send weibo failed!
		// ...
	}
}

</code></pre>

<h1>Q&A</h1>
If you have any question, or any suggestion, please mail to me.<br>
chenchao@staff.sina.com.cn or chengang4@staff.sina.com.cn