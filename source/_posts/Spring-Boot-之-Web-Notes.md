---
title: Spring Boot 之 Web Notes
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-31 14:50:41
categories: 开发者手册
tags: SpringBoot
---
一些Web的笔记！
![](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=259245079,1830956198&fm=26&gp=0.jpg)
<!--more-->
# 开发流程
1. Database Column
2. Model：模型定义，和数据库匹配
3. Dao：数据读取
4. Service：服务包装
5. Controller：业务入口
6. Test

# solr

官方网站： http://lucene.apache.org/solr/

...\bin> solr -e cloud -noprompt

搜索语法：

1. +包含关键词
2. -不需要某关键词

云模式solr start -e cloud –noprompt

单机solr start

solr create_collection wenda

关闭

 solr stop -all

核心概念：

Document：每个被索引的文档

Field：文档里的各个属性值

IKAnanlyzer分词配置（managed-schema）

```xml
<fieldTypename="text_ik" class="solr.TextField">

<!--索引时候的分词器-->

<analyzer type="index">

<tokenizerclass="org.wltea.analyzer.util.IKTokenizerFactory" useSmart=“false"/>

<filter class="solr.LowerCaseFilterFactory"/>

</analyzer>

<!--查询时候的分词器-->

<analyzer type="query">

<tokenizerclass="org.wltea.analyzer.util.IKTokenizerFactory" useSmart=“true"/>

</analyzer>

</fieldType>

```



```xml
<field name="question_title" type="text_ik" indexed="true" stored="true" multiValued="true"/>
<field name="question_content" type="text_ik" indexed="true" stored="true" multiValued="true"/>
```



数据库数据导入

```xml
solrconfig.xml

<requestHandlername="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">

<lstname="defaults">

<strname="config">data-config.xml</str>

</lst>

</requestHandler>

data-config.xml

<dataConfig>

<dataSource type="JdbcDataSource"

driver="com.mysql.jdbc.Driver"

url="jdbc:mysql://localhost/wenda"

user="root"

password="nowcoder"/>

<document>

<entity name="question" query="select id,title,content from question">

<field column="content" name="question_content"/>

<field column="title" name="question_title"/>

</entity>

</document>

</dataConfig>

```





***



# 敏感词
前缀树
• 根节点不包含字符，除根节点外每一个节点都只包含一个字符
• 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串
• 每个节点的所有子节点包含的字符都不相同

过滤掉符号
避免敏感词中间空格就识别不出了
```
@Service
public class SensitiveService implements InitializingBean {

    private static final Logger logger = LoggerFactory.getLogger(SensitiveService.class);

    /**
     * 默认敏感词替换符
     */
    private static final String DEFAULT_REPLACEMENT = "***";

    private class TrieNode {

        /**
         * true 关键词的终结 ； false 继续
         */
        private boolean end = false;

        /**
         * key下一个字符，value是对应的节点
         */
        private Map<Character, TrieNode> subNodes = new HashMap<>();

        /**
         * 向指定位置添加节点树
         */
        void addSubNode(Character key, TrieNode node) {
            subNodes.put(key, node);
        }

        /**
         * 获取下个节点
         */
        TrieNode getSubNode(Character key) {
            return subNodes.get(key);
        }

        boolean isKeywordEnd() {
            return end;
        }

        void setKeywordEnd(boolean end) {
            this.end = end;
        }

        public int getSubNodeCount() {
            return subNodes.size();
        }
    }

    /**
     * 根节点
     */
    private TrieNode rootNode = new TrieNode();


    /**
     * 判断是否是一个符号
     */
    private boolean isSymbol(char c) {
        int ic = (int) c;
        // 0x2E80-0x9FFF 东亚文字范围
        return !CharUtils.isAsciiAlphanumeric(c) && (ic < 0x2E80 || ic > 0x9FFF);
    }

    /**
     * 过滤敏感词
     */
    public String filter(String text) {
        if (StringUtils.isBlank(text)) {
            return text;
        }
        String replacement = DEFAULT_REPLACEMENT;
        StringBuilder result = new StringBuilder();

        TrieNode tempNode = rootNode;
        int begin = 0; // 回滚数
        int position = 0; // 当前比较的位置

        while (position < text.length()) {
            char c = text.charAt(position);
            // 空格直接跳过
            if (isSymbol(c)) {
                if (tempNode == rootNode) {
                    result.append(c);
                    ++begin;
                }
                ++position;
                continue;
            }

            tempNode = tempNode.getSubNode(c);

            // 当前位置的匹配结束
            if (tempNode == null) {
                // 以begin开始的字符串不存在敏感词
                result.append(text.charAt(begin));
                // 跳到下一个字符开始测试
                position = begin + 1;
                begin = position;
                // 回到树初始节点
                tempNode = rootNode;
            } else if (tempNode.isKeywordEnd()) {
                // 发现敏感词， 从begin到position的位置用replacement替换掉
                result.append(replacement);
                position = position + 1;
                begin = position;
                tempNode = rootNode;
            } else {
                ++position;
            }
        }

        result.append(text.substring(begin));

        return result.toString();
    }

    private void addWord(String lineTxt) {
        TrieNode tempNode = rootNode;
        // 循环每个字节
        for (int i = 0; i < lineTxt.length(); ++i) {
            Character c = lineTxt.charAt(i);
            // 过滤空格
            if (isSymbol(c)) {
                continue;
            }
            TrieNode node = tempNode.getSubNode(c);

            if (node == null) { // 没初始化
                node = new TrieNode();
                tempNode.addSubNode(c, node);
            }

            tempNode = node;

            if (i == lineTxt.length() - 1) {
                // 关键词结束， 设置结束标志
                tempNode.setKeywordEnd(true);
            }
        }
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        rootNode = new TrieNode();
//读取文本
        try {
            InputStream is = Thread.currentThread().getContextClassLoader()
                    .getResourceAsStream("SensitiveWords.txt");
            InputStreamReader read = new InputStreamReader(is);
            BufferedReader bufferedReader = new BufferedReader(read);
            String lineTxt;
            while ((lineTxt = bufferedReader.readLine()) != null) {
                lineTxt = lineTxt.trim();
                addWord(lineTxt);
            }
            read.close();
        } catch (Exception e) {
            logger.error("读取敏感词文件失败" + e.getMessage());
        }
    }

    public static void main(String[] argv) {
        SensitiveService s = new SensitiveService();
        s.addWord("色情");
        s.addWord("好色");
        System.out.print(s.filter("你好X色情XX"));
    }
}
```

# Web敏感词过滤
```
public int addQuestion(Question question) {
   //Html过滤
question.setTitle(HtmlUtils.htmlEscape(question.getTitle()));
question.setContent(HtmlUtils.htmlEscape(question.getContent()));
// 敏感词过滤
question.setTitle(sensitiveService.filter(question.getTitle()));
question.setContent(sensitiveService.filter(question.getContent()));
return questionDAO.addQuestion(question) > 0 ? question.getId() : 0;
    }
```

# 多线程
优势
•充分利用多处理器
•可以异步处理任务

挑战
•数据会被多个线程访问，有安全性问题
•不活跃的线程也会占用内存资源
•死锁
## Thread
1.extends Thread，重载run()方法
2.implements Runnable()，实现run()方法
```
new Thread(new Runnable() {
@Override
public void run() {
	Random random = new Random();
		for (int i = 0; i < 10; ++i) {
			sleep(random.nextInt(1000));
			System.out.println(String.format("T%d : %d", tid, i));
		}
	}
}, String.valueOf(i)).start();
```
## Synchronized－内置锁
1.放在方法上会锁住所有synchronized方法
2.synchronized(obj) 锁住相关的代码段
```
public static void testSynchronized1() {
synchronized (obj) {
Random random = new Random();
for (int i = 0; i < 10; ++i) {
sleep(random.nextInt(1000));
}
}
}
```
## BlockingQueue 同步队列

## ThreadLocal
1.线程局部变量。即使是一个static成员，每个线程访问的变量是不同的。
2.常见于web中存储当前用户到一个静态工具类中，在线程的任何地方都可以访问到当前线程的用户。
3.参考HostHolder.java里的users

## Executor
1.提供一个运行任务的框架。
2.将任务和如何运行任务解耦。
3.常用于提供线程池或定时任务服务
ExecutorService service = Executors.newFixedThreadPool(2);service.submit(new Runnable() {@Overridepublic void run() {for (int i = 0; i < 10; ++i) {sleep(1000);System.out.println("Execute %d" + i);}}});

## Future
线程间通信
1.返回异步结果
2.阻塞等待返回结果
3.timeout
4.获取线程中的Exception
```
public static void testFuture() {
ExecutorService service = Executors.newSingleThreadExecutor();
Future<Integer> future = service.submit(new Callable<Integer>() {
@Overridepublic Integer call() throws Exception {sleep(1000);
//throw new IllegalArgumentException("一个异常");
return 1;
}
});
service.shutdown();
try {
System.out.println(future.get());
//System.out.println(future.get(100, TimeUnit.MILLISECONDS));
} catch (Exception e) {
e.printStackTrace();
}
}
```

![](https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=3511180315,839779881&fm=26&gp=0.jpg)