## 基于 vux vue2的无限滚动插件

### 使用

#### 举个栗子

	<vScroller
            v-model="status"
            :use-pullup="true"
            :use-pulldown="true"
            @on-pulldown-loading="pullDown"
            @on-pullup-loading="pullUp"
            ref="scroller">
        <ul>
        	<li>1</li>
        	<li>2</li>
        	<li>3</li>
        	<li>4</li>
        	<li>5</li>
        	<li>6</li>
        </ul>
    </vScroller>
    <script>
    	import Scroller from 'PulldownLayer.vue';
    	export default {
    		components: {
            	Scroller,
        	},
        	data() {
        		return {
        			status: {
						upStatus: 'default',
						downStatus: 'default',
                	};
        		}
        	},
        	methods: {
        		pullDown() {
        			// do ajax
        	 		this.$refs.scroller.donePulldown(); // 加载完成时设置loading不可见
        		},
        		pullUp() {
        			// do ajax
        			if (没有更多数据了) {
        				this.$refs.scroller.disablePullup(); // 没有更多数据了 禁用上拉加载
        			}
        			this.$refs.scroller.donePullup(); // 加载完成时设置上拉loading不可见（只有设置了才能再次上拉）
        		},
        	},
    	}
    </script>
    
### API

### Props

| 参数      | 说明          | 类型      | 默认值  |可选值  |
|---------- |-------------- |---------- |-------- |------|
| model	  |上拉 下拉 加载状态, 底部展示文案| Object | 见详情 |
|usePullup|是否启用上拉加载|Boolean|false| |
|usePulldown|是否启用下拉加载|Boolean|false| |
|scrollOffset|触发距离|Number|60| |
|height|滚动容器高度|Number|100%| |
|loadingType|下拉加载loading样式|String| |'G7'|

`特别注意`： 如果滚动容器没有设置高度，那么scroller组件的父容器必须制定高度（100% 或 定值）

### model arguments

| 参数      | 说明          | 类型      | 默认值  |
|---------- |-------------- |---------- |-------- |
| upStatus|上拉状态| string | 'default'|
| downStatus|下拉状态| string | 'default'|
| pullupText|上拉时底部展示文案| string | ''|

### upStatus value 

| 值      | 说明          | 类型      |
|---------- |-------------- |---------- |
| 'up'|上拉状态-正在上拉中| string |
| 'loading'|上拉触发加载中| string |

### downStatus value 

| 值      | 说明          | 类型      |
|---------- |-------------- |---------- |
| 'down'|下拉状态-正在下拉中| string |
| 'loading'|下拉触发加载中| string |

### Methods

| method        | 说明           | 参数        | 参数说明        |
|------------|----------------|------------|------------|
|donePulldown|完成下拉，隐藏loading|null|（只有设置了才能再次上拉）|
|donePullup|完成上拉，隐藏loading|null|（只有设置了才能再次上拉）|
|disablePullup|禁用上拉|null|null|
|enablePullup|启用上拉|null|null|
|disablePulldown|禁用下拉|null|null|
|enablePulldown|启用下拉|null|null|
    
    