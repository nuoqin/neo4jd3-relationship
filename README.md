## neo4jd3-relationship

### 简介
本系统主要用于关系图谱展示。可以用于neo4j图数据库展示，也可以自定义数据集进行展示。
修改neo4jd3的部分代码，实现了以下的效果。
### 使用
1.安装d3
```
npm install d3@4.6.0 -S
```
2.在main.js配置neo4jd3.css全局样式
```
import './assets/d3/css/neo4jd3.css'
```
3.复制文件到任意vue2项目中
4.使用，请参考src下的文件（删除了对图标的支持）

#### 展示
![image](https://github.com/user-attachments/assets/c7f0494d-9e86-44bb-aaae-6ef09dddf087)

#### 鼠标右键
![image](https://github.com/user-attachments/assets/b77c3076-af18-4dd1-98ea-beeaae43e9f1)

#### 单击节点
![image](https://github.com/user-attachments/assets/53367250-b02f-49f5-a673-0f1e7d95ef45)
目前已集成到vue2的项目中。
开源不易，希望给一个start，谢谢啦
neo4jd3源项目地址在：https://github.com/eisman/neo4jd3
