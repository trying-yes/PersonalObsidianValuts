Python虚拟环境是一种创建隔离的Python运行环境的方法，它可以帮助你在不同的项目中使用不同的依赖版本，避免冲突。以下是Python虚拟环境的使用指南：

### 一、安装虚拟环境

Python虚拟环境可以通过`venv`模块（Python 3.3+自带）来创建。

1. **使用`venv`模块**
    
    - Python 3.3及以上版本自带`venv`模块，无需额外安装。它python用于创建虚拟环境。
        
    - 示例：
        
        ```python
        python -m venv myenv
        ```
        
        这会在当前目录下创建一个名为`myenv`的文件夹，里面包含虚拟环境的所有文件。

### 二、激活虚拟环境

创建虚拟环境后，需要激活它才能使用。

1. **在Windows系统中激活**
    
    - 在命令行中运行：
        
        ```bash
        你虚拟环境文件夹的名字（一般是.venv或者myenv）\Scripts\activate
        ```
        
    - 激活后，命令行提示符会显示虚拟环境的名称（如`(myenv)`）。
        
### 三、在虚拟环境中安装和管理依赖

1. **安装依赖**
    
    - 在虚拟环境激活状态下，使用`pip`安装依赖，这些依赖只会安装在虚拟环境中，不会影响全局Python环境。
        
        ```bash
        pip install requests
        ```
        
    - 可以通过`pip list`查看当前虚拟环境中安装的包。
        
2. **导出依赖**
    
    - 如果需要将虚拟环境中的依赖导出为`requirements.txt`文件，可以运行：
        
        ```bash
        pip freeze > requirements.txt
        ```
        
    - 这样可以方便地在其他机器或项目中复现相同的依赖环境。
        
3. **安装依赖文件**
    
    - 如果有`requirements.txt`文件，可以通过以下命令安装其中的依赖：
        
        ```bash
        pip install -r requirements.txt
        ```
        

### 四、退出虚拟环境

完成工作后，可以通过以下命令退出虚拟环境：

```bash
deactivate
```

退出后，命令行或终端会恢复到全局Python环境。
但是注意，只要你在使用，就必须激活
