---
title: 使用PageRank算法计算球队排名
date: 2017-07-08 11:27:00
tags: ['数据挖掘','足球','算法','Java']
mathjax: true
---
> 这是voidAlex原创的第一篇博文。
> 源码在[我的GitHub](https://github.com/voidAlex/pagerank)
<!-- more -->

## PageRank算法
PageRank算法，又叫佩奇排名。是由Google公司创始人拉里佩奇（Larry Page）发明的一种由搜索引擎根据网页之间相互的超链接计算的技术。

对于某个互联网网页A来说，该网页PageRank的计算基于以下两个基本假设：
>* 数量假设：在Web图模型中，如果一个页面节点接收到的其他网页指向的入链数量越多，那么这个页面越重要；
>* 质量假设：指向页面A的入链质量不同，质量高的页面会通过链接向其他页面传递更多的权重。所以越是质量高的页面指向页面A，则页面A越重要。

根据上面的两个假设，PageRank的计算步骤如下：
1. 网页通过链接关系构建起Web图，每个页面设置相同的PageRank值，通过若干轮的计算，会得到每个页面所获得的最终PageRank值。随着每一轮的计算进行，网页当前的PageRank值会不断得到更新。

2. 在一轮中更新页面PageRank得分的计算方法：在一轮更新页面PageRank得分的计算中，每个页面将其当前的PageRank值平均分配到本页面包含的出链上，这样每个链接即获得了相应的权值。而每个页面将所有指向本页面的入链所传入的权值求和，即可得到新的PageRank得分。当每个页面都获得了更新后的PageRank值，就完成了一轮PageRank计算。

## 使用PageRank算法计算球队实力
将PageRank算法应用到球队中后，球队的PageRank分数的计算依然基于两个假设：
>* 数量假设：比赛中A在其他球队身上取得的分数（战胜或战平）越多，那么这个球队实力越强；
>* 质量假设：取得积分的对手实力强弱不同，实力强的球队会提供更多的权重。所以A取得战胜或战平的球队实力越强，则球队A越强。

利用以上两个假设，PageRank算法刚开始赋予每个球队相同的重要性得分（PR值），通过迭代递归计算来更新每个球队节点的PageRank得分，直到得分稳定为止。

假设一个由4支球队组成的足球联赛：A，B，C和D。给定所有球队一个相同的初始PageRank值PR。在第一轮计算中，对于球队A，假设它在对阵B、C、D球队取得的积分分别为P1、P2、P3，那么它的PageRank值将被更新为：

$$ PR(A) = (PR(B)∙P1+PR(C)∙P2+PR(D)∙P3)/3 $$

同样的，B、C、D队的PageRank值也将通过此方法更新。在进行若干次迭代后，所有球队的PageRank值将会趋向于稳定，也就是收敛状态。这时所有球队的PageRank值就是它们的最终得分。

## 意义
灵感来源于虎扑的一个[帖子](https://bbs.hupu.com/18776888.html)，里面提到了球队的硬仗能力，或者球队上限。原贴的作者计算了上赛季的英超的PageRank分，自己看了之后比较感兴趣，就实现了一遍。原贴作者提到：
> *一支球队的联赛排名是其硬仗水平与虐菜能力的共同作用，然而由联赛排名决定的欧战资格，尤其是淘汰赛阶段，更看中的却是球队的硬仗水平，即话题区里所谓的球队上限。于是问题来了，是否存在有些球队主要靠虐菜能力进入欧战区，然后面对他国豪强一泻千里，给本国联赛拖了后腿的情况？我试着用PageRank算法来验证这一猜想。*

> *一支球队的PageRank评分都是从其他球队手中抢来的，要知道一支球队的PageRank评分就必须先知道其他球队的评分，这是鸡生蛋蛋生鸡的难题。PageRank算法的做法是给每个队一个初始分数，比如1，然后通过交战记录互相贡献分数，这样每支队的分数都会有变化；再拿这些分数重复一遍计算，每支队的分数又会变化；重复N次后，变化会趋于0（有数学证明），这时的分数就是最终结果。*

一支球队的所谓硬仗能力或球队上限是比较抽（xuan）象（xue）的东西，PageRank算法可以比较好的量化它。

本文中计算了本赛季（2016-2017赛季）英超、西甲、NBA各个球队的PageRank值。数据来源于[虎扑](https://soccer.hupu.com/table/)。

## 核心代码

核心代码有两部分，PageRank算法和爬虫。

### PageRank算法
用PageRank算法计算球队PageRank值比较简单。就是通过大量的交手记录来更新PageRank值。计算方法在第二部分已经说明。

用爬虫爬取的球队信息和比赛记录存放在文件中。球队信息以JSON的形式存放，比赛结果以文本的形式存放，一行表示一条比赛记录。类似于：
> 莱加内斯 2-4 皇马

这样的格式。

首先构造TeamItem类来存放球队信息，该类有两个字段：name和pagerank。

```Java
package com.pagerank.core;

/**
 * 球队类
 * Created by 王麟东 on 2017/7/8 0008.
 */
public class TeamItem {
    private String name;
    private double pagerank;

    public TeamItem(){
        this.pagerank = 1;
    }


    public String getName() {
        return name;
    }


    public double getPagerank() {
        return pagerank;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setPagerank(double pagerank) {
        this.pagerank = pagerank;
    }
}
```

然后构造MatchResult类来存放比赛结果。在MatchResult类中，有获取比赛结果的方法getWeight。根据比赛结果，返回对应的权重。

```Java
package com.pagerank.core;

/**
 * 比赛结果类
 * Created by 王麟东 on 2017/7/8 0008.
 */
public class MatchResult {
    private String teamA;
    private String teamB;
    private int scoreA;
    private int scoreB;

    public MatchResult(String line) {
        String temp[] = line.split("-");

        this.teamA = temp[0].split(" ")[0];
        this.scoreA = Integer.parseInt(temp[0].split(" ")[1]);

        this.teamB = temp[1].split(" ")[1];
        this.scoreB = Integer.parseInt(temp[1].split(" ")[0]);
    }

    /**
     * 获得权重（比赛结果）
     * @param team 球队名
     * @return 该球队在本条比赛记录中的比赛结果，胜3平1负0，若本条比赛记录中没有这个球队，返回-1
     */

    public int getWeight(String team){
        int weight = -1;
        if (team.equals(this.teamA)){
            if (this.scoreA == this.scoreB){
                weight = 1;
            }else if (this.scoreA > this.scoreB){
                weight = 3;
            }else if (this.scoreA < this.scoreB){
                weight = 0;
            }
        }else if (team.equals(this.teamB)){
            if (this.scoreB == this.scoreA){
                weight = 1;
            }else if (this.scoreB > this.scoreA){
                weight = 3;
            }else if (this.scoreB < this.scoreA){
                weight = 0;
            }
        }

        return weight;
    }

    /**
     * 在获得比赛权重不为-1的情况下，获得对手球队名
     * @param team 球队名
     * @return 本条比赛记录中的对手球队名
     */

    public String getOtherTeam(String team){
        String otherTeam = this.teamA;

        if (team.equals(this.teamA)){
            otherTeam = this.teamB;
        }

        return otherTeam;
    }
}
```
然后是PageRank类。在PageRank类中，构造方法首先将球队信息和比赛结果读取到内存中，并且给每个球队赋初始PageRank值1。

```Java
package com.pagerank.core;

import com.google.gson.Gson;
import com.google.gson.JsonArray;
import com.google.gson.JsonElement;
import com.google.gson.JsonObject;

import java.io.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Created by 王麟东 on 2017/7/8 0008.
 */
public class PageRank {
    private Map<String, TeamItem> teamMap;
    private List<MatchResult> matchResultList;
    private String teamInfopath;
    private String matchResultPath;
    private int max;

    public PageRank(String teamInfopath, String matchResultPath, int max){
        this.teamInfopath = teamInfopath;
        this.matchResultPath = matchResultPath;
        this.max = max;
        init();
    }

    /**
     * 初始化，将球队信息和比赛结果读取到内存
     */

    private void init(){
        try {
            BufferedReader teamReader = new BufferedReader(new FileReader(new File(this.teamInfopath)));
            Gson gson = new Gson();
            JsonArray jsonArray = gson.fromJson(teamReader.readLine(), JsonArray.class);
            teamReader.close();

            this.teamMap = new HashMap<String, TeamItem>();
            for (JsonElement jsonElement : jsonArray){
                TeamItem teamItem = new TeamItem();
                teamItem.setName(jsonElement.getAsJsonObject().get("team").getAsString());
                this.teamMap.put(teamItem.getName(), teamItem);
            }

            BufferedReader matchReader = new BufferedReader(new FileReader(new File(this.matchResultPath)));
            String line = "";

            this.matchResultList = new ArrayList<MatchResult>();
            while ((line = matchReader.readLine()) != null){
                MatchResult matchResult = new MatchResult(line);
                this.matchResultList.add(matchResult);
            }
            matchReader.close();
        }catch (IOException e){
            System.out.println("读取失败");
            System.exit(1);
        }

    }

    public Map<String, TeamItem> getTeamMap() {
        return teamMap;
    }

    public int getMax() {
        return max;
    }
}
```

初始化之后，开始递归的计算每个球队的PageRank值：

```Java
    /**
     * PageRank算法迭代器
     * @param teamMaps 球队信息
     * @param max 迭代次数
     */

    public void Iteration(Map<String, TeamItem> teamMaps, int max){
        Map<String, TeamItem> tmp = new HashMap<String, TeamItem>();

        for (TeamItem teamItem : teamMaps.values()){
            TeamItem tmpTeam = new TeamItem();
            tmpTeam.setName(teamItem.getName());
            double pagerank = 0;
            int count = 0;

            for (MatchResult matchResult : this.matchResultList){
                int weight = matchResult.getWeight(teamItem.getName());
                if (weight != -1){
                    double pr = teamMaps.get(matchResult.getOtherTeam(teamItem.getName())).getPagerank();
                    pagerank += (weight * pr);
                    count ++;
                }
            }

            tmpTeam.setPagerank(pagerank / count);
            tmp.put(tmpTeam.getName(), tmpTeam);
        }
        max --;
        this.teamMap = tmp;
        if (max > 0){
            Iteration(this.teamMap, max);
        }
    }
```

使用爬取的数据测试：
```Java
String team = "nba_team_list.json";
String match = "nba_result_list";
int max = 20;

PageRank pageRank = new PageRank(team, match, max);
pageRank.Iteration(pageRank.getTeamMap(), pageRank.getMax());
pageRank.wirteToFile();
pageRank.print();
```

由于数据量不大，迭代10-20次已经能够达到收敛状态。

### 爬虫
数据全部来源于虎扑，但是虎扑已经更新到了新赛季（2017-2018赛季）的数据，原来的爬虫失效，所以就不贴出来了。爬取的数据放在我的github。

## 结果
废话不多说，直接上结果。表格中的变化值为相比较原排名的变化程度。
### NBA
数据为2016-2017赛季常规赛的全部数据。

东部联盟

|球队 |PageRank |排名 |变化值 |
|----|----|----|----|
|凯尔特人 |1.9366 |1 |0 |
|猛龙 |1.8867 |2 |+1 |
|骑士 |1.8766 |3 |-1 |
|奇才 |1.7852 |4 |0 |
|老鹰 |1.6612 |5 |0 |
|公牛 |1.5974 |6 |+2 |
|热火 |1.5771 |7 |+2 |
|雄鹿 |1.5465 |8 |-2 |
|步行者 |1.5442 |9 |-2 |
|活塞 |1.3974 |10 |0 |
|黄蜂 |1.2995 |11 |0 |
|尼克斯 |1.1173 |12 |0 |
|魔术 |1.0865 |13 |0 |
|76人 |1.0001 |14 |0 |
|篮网 |0.6985 |15 |0 |

西部联盟

|球队 |PageRank |排名 |变化值 |
|----|----|----|----|
|勇士 |2.5695 |1 |0 |
|马刺 |2.3791 |2 |0 |
|火箭 |2.0306 |3 |0 |
|快船 |1.9787 |4 |0 |
|爵士 |1.8741 |5 |0 |
|雷霆 |1.7872 |6 |0 |
|灰熊 |1.7141 |7 |0 |
|开拓者 |1.5253 |8 |0 |
|掘金 |1.4822 |9 |0 |
|小牛 |1.2437 |10 |+1 |
|鹈鹕 |1.2392 |11 |-1 |
|国王 |1.1998 |12 |0 |
|森林狼 |1.1934 |13 |0 |
|湖人 |1.0070 |14 |0 |
|太阳 |0.9156 |15 |0 |

### 西甲
数据为2016-2017赛季西甲联赛的全部数据。

|球队 |PageRank |排名 |变化值 |
|----|----|----|----|
|巴萨 |2.3323 |1 |+1 |
|皇马 |2.2955 |2 |-1 |
|马竞 |1.8039 |3 |0 |
|塞维利亚 |1.7653 |4 |0 |
|比利亚雷亚尔 |1.6181 |5 |0 |
|毕尔巴鄂 |1.4819 |6 |+1 |
|阿拉维斯 |1.4205 |7 |+2 |
|皇家社会 |1.3817 |8 |-1 |
|埃瓦尔 |1.2816 |9 |+1 |
|西班牙人 |1.2111 |10 |-2 |
|马拉加 |1.1516 |11 |0 |
|瓦伦西亚 |1.1018 |12 |0 |
|塞尔塔 |0.9893 |13 |0 |
|拉斯帕尔马斯 |0.9532 |14 |0 |
|拉科鲁尼亚 |0.8892 |15 |+1 |
|皇家贝蒂斯 |0.8815 |16 |-1 |
|莱加内斯 |0.7624 |17 |0 |
|希洪竞技 |0.6651 |18 |0 |
|格拉纳达 |0.5086 |19 |+1 |
|奥萨苏纳 |0.4939 |20 |-1 |

### 英超
数据为2016-2017赛季英超联赛的全部数据。

|球队 |PageRank |排名 |变化值 |
|----|----|----|----|
|切尔西 |5.3667 |1 |0 |
|热刺 |4.9594 |2 |0 |
|利物浦 |4.7785 |3 |+1 |
|曼城 |4.2577 |4 |-1 |
|阿森纳 |4.1272 |5 |0 |
|曼联 |3.9255 |6 |0 |
|埃弗顿 |3.3458 |7 |0 |
|伯恩茅斯 |2.6264 |8 |+1 |
|莱斯特 |2.5141 |9 |+3 |
|南安普顿 |2.4357 |10 |-2 |
|西汉姆联 |2.4165 |11 |0 |
|水晶宫 |2.4097 |12 |+2 |
|西布朗 |2.3787 |13 |+3 |
|斯旺西 |2.3119 |14 |+1 |
|伯恩利 |2.2320 |15 |+1 |
|沃特福德 |2.2083 |16 |+1 |
|斯托克城 |2.1396 |17 |-4 |
|胡尔城 |1.9424 |18 |0 |
|米德尔斯堡 |1.4478 |19 |0 |
|桑德兰 |1.3255 |20 |0 |

至于结果能看出来什么信息，大家就见仁见智了。