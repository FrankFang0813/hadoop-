# hadoop-

* 修改本機hostname:
  ```js
  vi /etc/hostname
  ```
* 建立hadoop使用者以及群組(每台都要)  
  * 建立群組  
   ```js
   sudo addgroup hadoop_group  
   ```
   * 建立 Hadoop 專用帳戶
   ```js
   sudo adduser --ingroup hadoop_group [帳號]
   ```
   * 將 hadoop_admin 帳戶加入 sudo 權限
     ```js
     sudo vi /etc/sudoers
     ```
     下方新增:
     ```js
     hadoop_admin ALL=(ALL:ALL) ALL
     ```
  * 切換至剛剛建立好的 hadoop 管理帳號
    ```js
    su hadoop_admin
    ```
    
* 建立金鑰:
   ```js
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys   
   ```    
  * scp authorized_keys到各個slave內
  * 如果沒有.ssh資料夾，就先`ssh localhost` 再登出就會出現了
  
  
* 安裝java
  * 下載jdk1.8
  * 解壓縮
  ```js
  tar zxvf "java檔名"
  ```
  * 把JAVA資料夾移動到home目錄
  ```js
  mv java檔名 ~/
  ```
  * 設定 `~/.bashrc` 檔
    ```js
    #安裝JAVA後,給JAVA_HOME路徑
    export JAVA_HOME=/home/{username}/jdk1.8.0_251   
    export PATH=$JAVA_HOME/bin:$PATH            
    ```
  * 執行~/.bashrc檔 `source ~/.bashrc`
  * 測試 
  ```js 
  java -version
  ```  
  
* 安裝hadoop
  * 解壓縮
  ```js
  tar zxvf "hadoop檔案"
  ```
  * 移動到 home目錄
  ```js
  mv "hadoop檔案" ~/
  ```
  * 修改 `core-site.xml` 檔
  ```js
  vim core-site.xml
  ```
  * 在最下方:
  ```js
  <!-- Put site-specific property overrides in this file. -->
  # 刪除以下
  <configuration>
  </configuration>    
  ```
  *複製以下:
  ```js
  <configuration>
        <property>
                <name>fs.defaultFS</name>(設定HDFS登入位置，port號為9000)
                <value>hdfs://master:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>(保存臨時文件的位置,預設是/tmp/hadoop-hadoop)
                <value>file:/usr/local/hadoop/tmp</value>
                <description>Abase for other temporary directories.</descripti>
        </property>
   </configuration>  
     ```

   * 修改 `hdfs-site.xml` 檔
     ```js
     vim hdfs-site.xml
     ```
  * 貼上   
    <name>dfs.namenode.secondary.http-address</name>  主要是設定Name Node欲保存的Metadata之儲存目錄    
    <name>dfs.datanode.data.dir</name> Data Node欲保存的資料之儲存目錄  
    <name>dfs.replication</name>  儲存資料之副本數(datanode要幾個)
    ex:  
     ```js
     <configuration>
           <property>
                   <name>dfs.namenode.secondary.http-address</name>  #可有可無
                   <value>master:50090</value>
           </property>
           <property>
                   <name>dfs.namenode.name.dir</name>
                   <value>/home/hadoop/hdfs/namenode</value>   #注意路徑
           </property>
           <property>
                   <name>dfs.datanode.data.dir</name>
                   <value>/home/hadoop/hdfs/datanode</value>
           </property>
           <property>
                   <name>dfs.replication</name>
                   <value>3</value>
           </property>
     </configuration>
     ```
  * 修改`slave` 檔，加上想要有datanode的主機名稱 (若hadoop3版以上,讀取則為worker檔,非slave):
     ```js
     master # 通常主機只會有namenode,從機只有datanode
     slave1
     slave2
     ```
     
  * 並於/etc/hosts 修改各機器ip及主機名稱 (每台都要)
  ```js
  127.0.0.1       localhost
  192.xxx.x.xxx   master
  192.xxx.x.xxx   slave1
  192.xxx.x.xxx   slave2
  ```
  * 修改 `hadoop-env.sh` 檔，在底下添加:
    ```js
    export JAVA_HOME=/home/spark/jdk1.8.0_251  #JAVA路徑一定得貼上
    export PATH=$JAVA_HOME/bin:$PATH

    export HADOOP_HOME=/home/spark/hadoop-2.10.0 #可加可不加
    export PATH=$HADOOP_HOME/bin:$PATH

    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop  #可加可不加
    
    # hadoop 3版可能會遇到 start之後 可能出現 ERROR: Cannot set priority of datanode process 3340
    HADOOP_SHELL_EXECNAME=root  # 加上這段以root執行,預設為 HADOOP_SHELL_EXECNAME='hdfs'
    
    ```
  * 修改`~/.bashrc`檔，底下添加:(記得source)
    ```js
    export HADOOP_HOME=/home/spark/hadoop-2.10.0   #注意路徑
    export PATH=$HADOOP_HOME/bin:$PATH

    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    ```
  * 建立需要的資料夾(hadoop/tmp 還有hdfs/namenode、datanode)
  #可建可不建,格式化會自動建立(hadoop namenode format)
  ```js
  cd ~
  mkdir tmp
  mkdir hdfs

  cd hdfs
  mkdir namenode
  mkdir datanode
  ```
  * scp 整個hadoop資料夾、jave資料夾、`~/.bashrc` 檔到各個slave:(記得每台都要source)
  ```js
  scp -r hadoop-2.10.0 xxx@xxx.xxx.x.xxx:~
  ```
  * 格式化hadoop:
  ```js
  hadoop namenode -format
  ```
  * 啟動hadoop:
  ```js
  cd ~
  cd hadoop-2.10.0/sbin
  ./start-dfs.sh
  ```
* 安裝yarn:(可有可無)
  * 修改 `yarn-site.xml` 檔
  ```js
  <configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
  </configuration>
  ```
  * 修改 `mapred-site.xml.template`檔
  ```js
  <configuration>
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
  </configuration>
  ```
#參考資料:[https://medium.com/@sleo1104/hadoop-3-2-0-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8%E8%88%87%E4%BB%8B%E7%B4%B9-22aa183be33a](https://medium.com/@sleo1104/hadoop-3-2-0-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8%E8%88%87%E4%BB%8B%E7%B4%B9-22aa183be33a)
