## Android S
### wave2
#### 21服务器
```shell
git clone http://10.10.61.201:9090/baseline/repo/wave/repo.git

repo/repo init -u ssh://jiangruifeng@10.10.61.21/home/jiangruifeng/amlogic_sdk/s/wave2/sdk/manifest.git -m s-premium-20230830.xml 

.repo/repo/repo sync -j8
```

#### 25服务器
```shell
repo init -u ssh://git@source.amlogic.com/wm-s905x4-premium-s/platform/manifest.git -m s-premium-20230830.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git --reference=/home/zhuyuelin/workspace/wm/mirror/s/wave2_mirror

.repo/repo/repo sync -j8
```

#### 122服务器
```shell
repo init -u ssh://git@source.amlogic.com/wm-s905x4-premium-s/platform/manifest.git -m s-premium-20230830.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git --reference=/newhome/lv4/zhuyuelin/workspace/wm/mirror/s/wave2_mirror

.repo/repo/repo sync -j8
```

### wave3
#### 21服务器
```shell
git clone http://10.10.61.201:9090/baseline/repo/wave/repo.git

repo/repo init -u ssh://jiangruifeng@10.10.61.21/home/jiangruifeng/amlogic_sdk/s/wave3/sdk/manifest.git -m google_gretzky_sdmc_wave3.xml

.repo/repo/repo sync -j8
```

#### 25服务器
```shell
repo init -u ssh://git@source.amlogic.com/bvs-s905y4-s/platform/manifest.git -m google_gretzky_sdmc_wave3.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git --reference=/home/zhuyuelin/workspace/wm/mirror/s/wave3_mirror

```

#### 122服务器
```shell
repo init -u ssh://git@source.amlogic.com/bvs-s905y4-s/platform/manifest.git -m google_gretzky_sdmc_wave3.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git --reference=/newhome/lv4/zhuyuelin/workspace/wm/mirror/S_wave3_mirror

.repo/repo/repo sync -j8
```

## Android U
### wave0
#### 21服务器
```shell
git clone http://10.10.61.201:9090/baseline/repo/wave/repo.git

repo/repo init -u ssh://jiangruifeng@10.10.61.21/home/jiangruifeng/amlogic_sdk/u/wm-0612/sdk/manifest.git -b master -m wm-coffey-20240612.xml

.repo/repo/repo sync -j8
```

#### 122服务器
```shell
repo init -u ssh://git@source.amlogic.com/wm-s905x5m-coffey-u/platform/manifest.git -m wm-coffey-20240612.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git --reference=/newhome/lv4/zhuyuelin/workspace/wm/mirror/u/wave0_mirror

.repo/repo/repo sync -j8
```

#### u wave0 s2u  
```shell  
git clone [http://10.10.61.201:9090/baseline/repo/wave/repo.git](http://10.10.61.201:9090/baseline/repo/wave/repo.git)  
  
repo/repo init -u ssh://jiangruifeng@10.10.61.21/home/jiangruifeng/amlogic_sdk/u/wm-0906/sdk/manifest.git -b master -m wm-wave0-20240906-s-u.xml  
.repo/repo/repo sync -j8  
```


### wave1 S-U
AML
```shell
repo init -u ssh://git@source.amlogic.com/wm-u-wave1/platform/manifest.git -m u-wave1-20241014.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git
.repo/repo/repo sync -j8 
```

21
```
git clone http://10.10.61.201:9090/baseline/repo/wave/repo.git
  
repo/repo init -u ssh://jiangruifeng@10.10.61.21/home/jiangruifeng/amlogic_sdk/u/wm-241014/manifest.git -b master -m u-wave1-20241014.xml  
.repo/repo/repo sync -j8
```


repo sync -c common/common14-5.15/common

repo manifest | grep “common”


.repo/repo/repo forall -c 'git clean -f -d'     # Clean untracked files

.repo/repo/repo forall -c 'git reset --hard'    # Remove all working directory (and staged) changes.  

.repo/repo/repo forall -c 'git clean -f -d'     # Clean untracked files


key仓库
http://10.10.61.201:9090/baseline/sdmc/android_u/keys/-/tree/release/