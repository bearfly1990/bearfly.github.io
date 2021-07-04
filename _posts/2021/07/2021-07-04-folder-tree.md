---
layout: post
title: Folder Tree Info DFS/BFS
subtitle: Iterate folder by DFS and BFS
date: 2021-07-04
author: BF
header-img: img/bf/python.jpg
catalog: true
tags:
  - python
  - algorithm
---

# 深度优先与广度优先遍历文件（夹）信息

最近做了一个对系统目录管理进行管理的页面，涉及到了对目录中的文件夹和文件的信息读取的问题，刚好回顾了一下文件夹遍历的算法。

由于需要在页面中体现层级目录关系，需要返回的json中有`parent_id`这样标志符，没有现成的库可以提供，便需要自己写一个。

当时第一个想法就是用递归的方式来遍历，就可以得到对应的关系，但是同一目录的文件（夹）的id就不能连续了，可以使用广度优先遍历。

## 目录结构
新建了一个测试的目录，放了一些文件：
![tree](/img/post/2021/07/2020-07-04-Folder-DFS-BFS-01-tree.gif)

下面是更直观的文件的关系示例：
![tree map](/img/post/2021/07/2020-07-04-Folder-DFS-BFS-02-tree-map.png)

## Depth-First-Search(DFS)

使用递归是最直接的深度优先搜索的方式，直接贴代码如下：
```python
import os
def iterate_folder_files_info_dfs(current_folder, file_list, root_id, level):
    level = level + 1
    current_id = root_id + 1
    files_in_folder = os.listdir(current_folder)
    for file_name in files_in_folder:
        file_full_path = os.path.join(current_folder, file_name)
        if os.path.isfile(file_full_path):
            new_file_info = {
                'id': current_id,
                'parent_id': root_id,
                'level': level,
                'type': 'file',
                'folder_name':current_folder,
                'file_name':file_name,
                'full_path':file_full_path,
                'file_size':os.path.getsize(file_full_path),
                'create_time':os.path.getctime(file_full_path)
            }
            file_list.append(new_file_info)
            current_id = current_id + 1
        elif os.path.isdir(file_full_path):
            new_folder_info = {
                'id': current_id,
                'parent_id': root_id,
                'level': level,
                'type': 'folder',
                'folder_name':current_folder,
                'file_name':file_name,
                'full_path':file_full_path,
                'create_time':os.path.getctime(file_full_path)
            }
            file_list.append(new_folder_info)
            current_id = iterate_folder_files_info_dfs(file_full_path, file_list, current_id, level)
    return current_id

file_list = []
current_folder = r'C:\test'
iterate_folder_files_info_dfs(current_folder, file_list, -1, 0)
with open('result.dfs.json','w+') as f:
    f.write(str(file_list))
```

逻辑还是比较直观的，一旦搜索到的是文件，就直接把信息加到最后的list中。如果是目录，把信息加到最后的list中后，继续向下调用搜索，只要最后一层没有目录了，就逐级向上返回自己的id。

最后生成的id如下图所示：
![dfs result](/img/post/2021/07/2020-07-04-Folder-DFS-BFS-03-map-dfs.png)
可以看到ID的分布是按上到下生成（按深度）生成的，下面是最后生成的json, parent_id也没有问题。
```javascript
[{
        'id': 0,
        'parent_id': -1,
        'level': 1,
        'type': 'folder',
        'folder_name': 'C:\\test',
        'file_name': 'classes',
        'full_path': 'C:\\test\\classes',
        'create_time': 1625323109.1690984
    }, {
        'id': 1,
        'parent_id': 0,
        'level': 2,
        'type': 'file',
        'folder_name': 'C:\\test\\classes',
        'file_name': 'class_introduce.txt',
        'full_path': 'C:\\test\\classes\\class_introduce.txt',
        'file_size': 23,
        'create_time': 1625323181.7119198
    }, {
        'id': 2,
        'parent_id': -1,
        'level': 1,
        'type': 'folder',
        'folder_name': 'C:\\test',
        'file_name': 'scores',
        'full_path': 'C:\\test\\scores',
        'create_time': 1625323098.449873
    }, {
        'id': 3,
        'parent_id': 2,
        'level': 2,
        'type': 'folder',
        'folder_name': 'C:\\test\\scores',
        'file_name': 'english',
        'full_path': 'C:\\test\\scores\\english',
        'create_time': 1625323396.2220247
    }, {
        'id': 4,
        'parent_id': 3,
        'level': 3,
        'type': 'file',
        'folder_name': 'C:\\test\\scores\\english',
        'file_name': 'english1.txt',
        'full_path': 'C:\\test\\scores\\english\\english1.txt',
        'file_size': 23,
        'create_time': 1625323470.866679
    }, {
        'id': 5,
        'parent_id': 3,
        'level': 3,
        'type': 'file',
        'folder_name': 'C:\\test\\scores\\english',
        'file_name': 'english2.txt',
        'full_path': 'C:\\test\\scores\\english\\english2.txt',
        'file_size': 55,
        'create_time': 1625323470.8686738
    }, {
        'id': 6,
        'parent_id': 2,
        'level': 2,
        'type': 'folder',
        'folder_name': 'C:\\test\\scores',
        'file_name': 'math',
        'full_path': 'C:\\test\\scores\\math',
        'create_time': 1625323316.5491896
    }, {
        'id': 7,
        'parent_id': 6,
        'level': 3,
        'type': 'file',
        'folder_name': 'C:\\test\\scores\\math',
        'file_name': 'math1.txt',
        'full_path': 'C:\\test\\scores\\math\\math1.txt',
        'file_size': 33,
        'create_time': 1625323454.2487762
    }, {
        'id': 8,
        'parent_id': 6,
        'level': 3,
        'type': 'file',
        'folder_name': 'C:\\test\\scores\\math',
        'file_name': 'math2.txt',
        'full_path': 'C:\\test\\scores\\math\\math2.txt',
        'file_size': 31,
        'create_time': 1625323460.5313218
    }, {
        'id': 9,
        'parent_id': 2,
        'level': 2,
        'type': 'file',
        'folder_name': 'C:\\test\\scores',
        'file_name': 'score_introduce.txt',
        'full_path': 'C:\\test\\scores\\score_introduce.txt',
        'file_size': 26,
        'create_time': 1625323421.2782009
    }, {
        'id': 10,
        'parent_id': -1,
        'level': 1,
        'type': 'file',
        'folder_name': 'C:\\test',
        'file_name': 'summary.txt',
        'full_path': 'C:\\test\\summary.txt',
        'file_size': 20,
        'create_time': 1625323064.0133853
    }
]
```
## Breadth-First-Search(BFS)
下面是使用广度优先算法的解决方案，代码如下：
```python
import os
def iterate_folder_files_info_bfs(current_folder, file_info_list, root_id, level):
    level = level + 1
    current_id = root_id + 1
    files_in_folder = os.listdir(current_folder)
    search_list = []
    level_files_list = []
    for file_name in files_in_folder:
        file_full_path = os.path.join(current_folder, file_name)
        level_files_list.append(file_full_path)
    search_list.append({root_id:level_files_list})
    
    while(search_list):
        level_files_map = search_list.pop(0)
        root_id = list(level_files_map.keys())[0]
        level_files_list = list(level_files_map.values())[0]
        for file_full_path in level_files_list:
            level_files_list_new = []
            if os.path.isfile(file_full_path):
                new_file_info = {
                    'id': current_id,
                    'parent_id': root_id,
                    'level': level,
                    'type': 'file',
                    'folder_name':os.path.dirname(file_full_path),
                    'file_name':os.path.basename(file_full_path),
                    'full_path':file_full_path,
                    'file_size':os.path.getsize(file_full_path),
                    'create_time':os.path.getctime(file_full_path)
                }
                file_info_list.append(new_file_info)
                current_id = current_id + 1
            elif os.path.isdir(file_full_path):
                new_folder_info = {
                    'id': current_id,
                    'parent_id': root_id,
                    'level': level,
                    'type': 'folder',
                    'folder_name':os.path.dirname(file_full_path),
                    'file_name':os.path.basename(file_full_path),
                    'full_path':file_full_path,
                    'create_time':os.path.getctime(file_full_path)
                }
                file_info_list.append(new_folder_info)
                for sub_file_name in os.listdir(file_full_path):
                     new_file_full_path = os.path.join(file_full_path, sub_file_name)
                     level_files_list_new.append(new_file_full_path)
                search_list.append({current_id:level_files_list_new})
                current_id = current_id + 1
                
        level = level +1


file_info_list = []
current_folder = r'C:\test'
iterate_folder_files_info_bfs(current_folder, file_info_list,  -1, 0)
with open('result.bfs.json','w+') as f:
    f.write(str(file_info_list))
```

逻辑相对于DFS稍微多了一些状态的维护，初始化第一层文件和文件夹信息后，把level和名字列表组成{level:[filename, foldername...]} 的dict加入遍历状态list。然后一直循环遍历list,把level信息拿出来，一旦搜索到的是文件，就直接把信息加到最后的list中，如果是目录，把文件信息和Level重新加回list，直到`search_list`为空。

最后生成的id如下图所示：
![dfs result](/img/post/2021/07/2020-07-04-Folder-DFS-BFS-03-map-bfs.png)
可以看到ID的分布是按左到右生成（按广度）生成的，下面是最后生成的json, parent_id也没有问题。

```javascript
[{
        'id': 0,
        'parent_id': -1,
        'level': 1,
        'type': 'folder',
        'folder_name': 'C:\\test',
        'file_name': 'classes',
        'full_path': 'C:\\test\\classes',
        'create_time': 1625323109.1690984
    }, {
        'id': 1,
        'parent_id': -1,
        'level': 1,
        'type': 'folder',
        'folder_name': 'C:\\test',
        'file_name': 'scores',
        'full_path': 'C:\\test\\scores',
        'create_time': 1625323098.449873
    }, {
        'id': 2,
        'parent_id': -1,
        'level': 1,
        'type': 'file',
        'folder_name': 'C:\\test',
        'file_name': 'summary.txt',
        'full_path': 'C:\\test\\summary.txt',
        'file_size': 20,
        'create_time': 1625323064.0133853
    }, {
        'id': 3,
        'parent_id': 0,
        'level': 2,
        'type': 'file',
        'folder_name': 'C:\\test\\classes',
        'file_name': 'class_introduce.txt',
        'full_path': 'C:\\test\\classes\\class_introduce.txt',
        'file_size': 23,
        'create_time': 1625323181.7119198
    }, {
        'id': 4,
        'parent_id': 1,
        'level': 3,
        'type': 'folder',
        'folder_name': 'C:\\test\\scores',
        'file_name': 'english',
        'full_path': 'C:\\test\\scores\\english',
        'create_time': 1625323396.2220247
    }, {
        'id': 5,
        'parent_id': 1,
        'level': 3,
        'type': 'folder',
        'folder_name': 'C:\\test\\scores',
        'file_name': 'math',
        'full_path': 'C:\\test\\scores\\math',
        'create_time': 1625323316.5491896
    }, {
        'id': 6,
        'parent_id': 1,
        'level': 3,
        'type': 'file',
        'folder_name': 'C:\\test\\scores',
        'file_name': 'score_introduce.txt',
        'full_path': 'C:\\test\\scores\\score_introduce.txt',
        'file_size': 26,
        'create_time': 1625323421.2782009
    }, {
        'id': 7,
        'parent_id': 4,
        'level': 4,
        'type': 'file',
        'folder_name': 'C:\\test\\scores\\english',
        'file_name': 'english1.txt',
        'full_path': 'C:\\test\\scores\\english\\english1.txt',
        'file_size': 23,
        'create_time': 1625323470.866679
    }, {
        'id': 8,
        'parent_id': 4,
        'level': 4,
        'type': 'file',
        'folder_name': 'C:\\test\\scores\\english',
        'file_name': 'english2.txt',
        'full_path': 'C:\\test\\scores\\english\\english2.txt',
        'file_size': 55,
        'create_time': 1625323470.8686738
    }, {
        'id': 9,
        'parent_id': 5,
        'level': 5,
        'type': 'file',
        'folder_name': 'C:\\test\\scores\\math',
        'file_name': 'math1.txt',
        'full_path': 'C:\\test\\scores\\math\\math1.txt',
        'file_size': 33,
        'create_time': 1625323454.2487762
    }, {
        'id': 10,
        'parent_id': 5,
        'level': 5,
        'type': 'file',
        'folder_name': 'C:\\test\\scores\\math',
        'file_name': 'math2.txt',
        'full_path': 'C:\\test\\scores\\math\\math2.txt',
        'file_size': 31,
        'create_time': 1625323460.5313218
    }
]
```

## 最后

深度优先搜索可以利用递归，而广度优先则需要维护好遍历的顺序，以便同一层搜索完后再向下一层搜索。

