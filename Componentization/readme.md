# 一个关于React的Componentized（组件化）的例子

01/11/2017

English version is on the way...

首先，显示效果如下图

![](1.png)

最早的核心部分的代码是这样的。

```javascript
// detail-card-component.js
// 很多其他components...
{
    this.props.offer.offer_type === TYPE_CROWDFUNDING ?
    <div style={styles.row2}>
        <div style={styles.box.bordered}>
            <p style={styles.title}>最低起投金额</p>
            {this.props.offer.min_investment == null ? "暂无资料" : `$${this.props.offer.min_investment}`}
        </div>
        <div style={styles.box.bordered}>
            <p style={styles.title}>创造就业总数／单笔</p>
            {this.props.offer.job_creation_per_investor && this.props.offer.job_creation_total
                ?
                `${this.props.offer.job_creation_total}/${this.props.offer.job_creation_per_investor}`
                :
                "暂无资料"}
        </div>
        <div style={styles.box.bordered}>
            <p style={styles.title}>银行贷款通过</p>
            {this.props.offer.loan_approve_status ? "通过" : "不通过"}
        </div>
        <div style={styles.box.bordered}>
            <p style={styles.title}>移民文件</p>
            {(this.props.offer.offer_526_status == true || this.props.offer.regional_center_526_status == true)
                ?
                (this.props.offer.regional_center_829_status == true)
                    ?
                    "I526 | I829"
                    :
                    "I526"
                :
                "暂无资料"
            }
        </div>
        <div style={styles.box}>
            <p style={styles.title}>投资类型</p>
            {this.props.offer.investment_type == null ? "暂无资料" : `${this.props.offer.investment_type}`}
        </div>
    </div>
    :
    <div style={styles.row2}>
        <div style={styles.box.bordered}>
            <p style={styles.title}>投资周期</p>
            {this.props.offer.term_months == null ? "暂无资料" : `${this.props.offer.term_months}年`}
        </div>
        <div style={styles.box.bordered}>
            <p style={styles.title}>最低起投金额</p>
            {this.props.offer.term_months == null ? "暂无资料" : `${this.props.offer.term_months}年`}
        </div>
        <div style={styles.box.bordered}>
            <p style={styles.title}>投资类型</p>
            {this.props.offer.investment_type ? this.props.offer.investment_type : "暂无资料"}
        </div>
        <div style={styles.box.bordered}>
            <p style={styles.title}>年均回报</p>
            {this.props.offer.investment_type ? this.props.offer.investment_type : "暂无资料"}
        </div>
        <div style={styles.box}>
            <p style={styles.title}>融资总额</p>
            {this.props.offer.total_capital_amount == null ? "暂无资料" : `$${this.props.offer.total_capital_amount}`}
        </div>
    </div>
}
// 很多其他components...
```

看完之后，五个感想：

1. 一脸懵逼，代码好多好复杂的样子。
2. 咦？好像有很多重复代码？
3. 文字和代码全部混杂在一起，感觉完全没有任何维护性。
4. 这跟特码HTML写出来的有什么区别你告诉我？！
5. 这特码怎么重用你告诉我！？

于是，改！

思路如下：【根据组件化思想】把所有数据提取出来放入上一层的container=>把React模板写好=>数据注入=>完成。

【提取数据到上一层】：

```javascript
// detail-card-container.js

_prepareData(offer) {
	return offer.offer_type === TYPE_CROWDFUNDING ? [{
        name: "最低起投金额",
        value: offer.min_investment == null ? "暂无资料" : `$${offer.min_investment}`
    }, {
        name: "创造就业总数／单笔",
        value: offer.job_creation_per_investor && offer.job_creation_total ?
            `${offer.job_creation_total}/${offer.job_creation_per_investor}`
            :
            "暂无资料"
    }, {
        name: "银行贷款通过",
        value: offer.loan_approve_status ? "通过" : "不通过"
    }, {
        name: "移民文件",
        value: (offer.offer_526_status == true || offer.regional_center_526_status == true) ?
            (offer.regional_center_829_status == true) ?
                "I526 | I829"
                :
                "I526"
            :
            "暂无资料"
    }, {
        name: "投资类型",
        value: offer.investment_type == null ? "暂无资料" : `${offer.investment_type}`
    }]
    :
    [{
        name: "投资周期",
        value: offer.term_months == null ? "暂无资料" : `${offer.term_months}年`
    }, {
        name: "最低起投金额",
        value: offer.min_investment == null ? "暂无资料" : `$${offer.min_investment}`
    }, {
        name: "投资类型",
        value: offer.investment_type ? offer.investment_type : "暂无资料"
    }, {
        name: "年均回报",
        value: offer.expected_return ? `$${offer.expected_return}` : "暂无资料"
    }, {
        name: "融资总额",
        value: offer.total_capital_amount == null ? "暂无资料" : `$${offer.total_capital_amount}`
    }];
    }
}

render() {
  	const items = _prepareData(this.props.offer);
  	return (
    	// 很多components...
      	<DetailCardComponent rowItems={items}/>
      	// 很多components...
    )
}
```

【重写React模板】：新建一个Component

```javascript
// card-profile-row.js
class CardProfileRow extends Component {
    render() {
        const items = this.props.items.map((item, i) => 
            <div key={i.toString()} style={this.props.items.length === i + 1 ? styles.box : styles.box.bordered}>
                <p style={styles.title}>{item.name}</p>
                {item.value}
            </div>
        );
        return (
            <div style={styles.cardProfileRow}>
                {items}
            </div>
        );
    }
}

export default CardProfileRow;
```

【数据注入】

```javascript
// detail-card-component.js
// 很多其他components...
<CardProfileRow items={this.props.rowItems} style={styles} />
// 很多其他components...
```

至此，我们完成了：

- [x] 一脸懵逼，代码好多好复杂的样子。=>实际代码量减少一半左右
- [x] 咦？好像有很多重复代码？=>没有重复代码
- [x] 文字和代码全部混杂在一起，感觉完全没有任何维护性。=>数据和代码分离，可维护
- [x] 这跟特码HTML写出来的有什么区别你告诉我？！=>这是React了！
- [x] 这特码怎么重用你告诉我！？=>现在至少可以重用成下图了！我们可以通过改变数据来重用这个组件而不需要修改任何代码，甚至数据的数量也可以改变！

![](2.png)

是的，解决了前人的遗留问题之后，又来了新需求，如果想用重用这个组件实现下面的界面，怎么做？！

![](3.png)

Excited!其实就是增加“多行显示”这个功能就行了，我们可以稍微改变一下数据格式，让数据由数组变成一个矩阵，也就是一维变成二维。

然后在这个过程中，我又非常偶然地发现，React居然支持pure functional component！之前还在想，component本身不知道自己需要哪些数据，在编程时带来了极大的不便，现在我们可以通过函数参数的方式提示自己这个component拥有哪些数据，需要哪些数据了！所以我们稍加修改：

```javascript
// detail-card-container.js

_prepareData(offer) {
	// 用[]把数据再包一层把它变成二维！
  	return [offer.offer_type === TYPE_CROWDFUNDING ? [{
        name: "最低起投金额",
        value: offer.min_investment == null ? "暂无资料" : `$${offer.min_investment}`
    }, {
        name: "创造就业总数／单笔",
        value: offer.job_creation_per_investor && offer.job_creation_total ?
            `${offer.job_creation_total}/${offer.job_creation_per_investor}`
            :
            "暂无资料"
    }, {
        name: "银行贷款通过",
        value: offer.loan_approve_status ? "通过" : "不通过"
    }, {
        name: "移民文件",
        value: (offer.offer_526_status == true || offer.regional_center_526_status == true) ?
            (offer.regional_center_829_status == true) ?
                "I526 | I829"
                :
                "I526"
            :
            "暂无资料"
    }, {
        name: "投资类型",
        value: offer.investment_type == null ? "暂无资料" : `${offer.investment_type}`
    }]
    :
    [{
        name: "投资周期",
        value: offer.term_months == null ? "暂无资料" : `${offer.term_months}年`
    }, {
        name: "最低起投金额",
        value: offer.min_investment == null ? "暂无资料" : `$${offer.min_investment}`
    }, {
        name: "投资类型",
        value: offer.investment_type ? offer.investment_type : "暂无资料"
    }, {
        name: "年均回报",
        value: offer.expected_return ? `$${offer.expected_return}` : "暂无资料"
    }, {
        name: "融资总额",
        value: offer.total_capital_amount == null ? "暂无资料" : `$${offer.total_capital_amount}`
    }]];
    }
}

render() {
  	const items = _prepareData(this.props.offer);
  	return (
    	// 很多components...
      	<DetailCardComponent rowItems={items}/>
      	// 很多components...
    )
}
```

```javascript
// card-profile-row.js
import React from 'react';

// 函数式编程加上尖头函数简直优美得想哭。。。
// 现在我们在这里声明什么这个component才能用什么，我们可以非常清楚地知道这个component拥有哪些数据，也可以很清楚地知道上一层需要提供哪些数据，而component自己完全dumb！它完全不知道数据来自哪，是什么，它只知道显示数据，这个单一职责的感觉太赞了！！！
export default ({ rows, styles }) => {
    console.log(rows);
    return (<div>
        {rows.map((row, i) =>
            <div key={i + 1000} style={styles.cardProfileRow}>
                {row.map((item, j) =>
                    <div key={j} style={row.length === j + 1 ? styles.box : styles.box.bordered}>
                        <p style={styles.title}>{item.name}</p>
                        {item.value}
                    </div>
                )}
            </div>
        )}
    </div>);
}
```

其他地方不需要修改（了吧？我应该没忘啥···）。



以上。