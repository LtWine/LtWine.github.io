---
layout:     post
title:      "Deploy Springboot With Python3"
subtitle:   "As a reference for the springboot project"
date:       2019-05-09 00:00:00
author:     "Wine"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - tech
    - springboot
	- python
    - Meta
---

```python
# Program: springboot项目部署
# Data: 2019-05-09
# By: liumeijian.github.io
import datetime
import os
import stat
import re
import shutil
import sys
import tarfile
import zipfile

# 必须在用户目录下新建Releases目录，把此脚本放到下面
# 项目目录 ~/app/application/ 其中app要先自己手动创建
# 使用python3.7环境运行
# 自用 仅做参考

# Home目录
HomePath = os.path.expanduser('~')
HomePath = HomePath + "/"
# Realease目录
ReleasePath = os.path.expanduser(HomePath + 'Releases/')
allFileToDeploy = []
dirName = ''
dirPath = ''
isbackUp = True
backUpDirName = '/backup'


def movefile(srcfile, dstfile):
    fpath, fname = os.path.split(dstfile)  # 分离文件名和路径
    if not os.path.exists(fpath):
        os.makedirs(fpath)  # 创建路径
    shutil.move(srcfile, dstfile)  # 移动文件


def makeBakDir(application_path):
    if not os.path.exists(application_path + backUpDirName):
        os.makedirs(application_path + backUpDirName)
    return application_path + backUpDirName


def tar(fname, fpath):
    t = tarfile.open(fname + ".tar", "w")
    for root, dir, files in os.walk(fpath):
        for file in files:
            fullpath = os.path.join(root, file)
            if backUpDirName not in fullpath and "/log/" not in fullpath and "/logs/" not in fullpath:
                t.add(fullpath)
    t.close()


def un_zip(zipFileName, target):
    f = zipfile.ZipFile(ReleasePath + zipFileName + ".zip", 'r')
    for file in f.namelist():
        f.extract(file, target)
    f.close()


def exit(errorMsg):
    print(errorMsg)
    sys.exit(1)


#### 获取所有需要部署的项目到allFileToDeploy下
def allFileToDeployM():
    global allFileToDeploy
    global dirName
    global dirPath
    # 所有在Release目录下的zip文件
    allfiles = os.listdir(ReleasePath)
    allZipFiles = [re.sub(r".zip", "", f) for f in allfiles if re.search('.zip$', f)]
    if len(sys.argv) <= 1:
        exit("参数错误 无法部署")

    # 全部署  python deploy.py
    if len(sys.argv) == 2:
        if not os.path.isdir(HomePath + sys.argv[1]):
            exit("请输入正确的应用部署目录")
        dirName = sys.argv[1]
        dirPath = HomePath + sys.argv[1]
        allFileToDeploy = allZipFiles
        # asw = input('是否在目录:%s 部署所有应用(y:n)\n'%(dirName))
        # if (asw != 'y'):
        #     exit("停止部署")

    # 部署目录和项目 python deploy.py app hic-ump  || python deploy.py cloud eureka-config
    if len(sys.argv) >= 3:
        if not os.path.isdir(HomePath + sys.argv[1]):
            exit("请输入正确的应用部署目录")
        else:
            dirName = sys.argv[1]
            dirPath = HomePath + sys.argv[1]
            for i in range(len(sys.argv)):
                if (i >= 2):
                    if sys.argv[i] in allZipFiles:
                        allFileToDeploy.append(sys.argv[i])
                    else:
                        print("不存在应用:" + sys.argv[i] + ".zip")

    print("所有即将部署的应用:", allFileToDeploy)
    if len(allFileToDeploy) == 0:
        exit("没有需要部署的项目")
    # asw = input("是否开始部署(y:n)\n")
    # if asw == 'y':
    #     print("开始部署")
    # else:
    #     exit("停止部署")


# 开始部署 备份
def deployApplication():
    for application in allFileToDeploy:
        print("开始部署目录:", dirName, "下的:", application, "项目")
        # 全量部署
        dir_path = HomePath + dirName + '/'
        application_path = dir_path + application
        if not os.path.isdir(application_path):
            un_zip(application, dir_path)
            os.chdir(application_path)
            os.chmod(application_path + "/start.sh",stat.S_IRWXU)
            os.system(application_path + "/start.sh > nohup.out 2>&1 &")
        else:
            # 开始备份
            os.chdir(application_path)
            os.chmod(application_path + "/stop.sh",stat.S_IRWXU)
            os.system(application_path + "/stop.sh")
            os.chdir(makeBakDir(application_path))
            if isbackUp:
                otherStyleTime = datetime.datetime.now().strftime("%Y%m%d%H%M%S")
                tar(application + otherStyleTime, application_path)
            # 增量部署
            un_zip(application, HomePath + 'Releases/')
            application_release_path = HomePath + 'Releases/' + application + '/'
            if os.path.isfile(application_release_path + application + '.war'):
                if os.path.isfile(application_path + application + '.war'):
                    os.remove(application_path + "/" + application + ".war")
                shutil.copy(application_release_path + application + '.war',
                            application_path + "/" + application + ".war")
            else:
                if os.path.isfile(application_path + application + '.jar'):
                    os.remove(application_path + "/" + application + ".jar")
                shutil.copy(application_release_path + application + '.jar',
                            application_path + "/" + application + ".jar")
                if os.path.isdir(application_path + "/lib"):
                    shutil.rmtree(application_path + "/lib", True)
                shutil.copytree(application_release_path + '/lib', application_path + "/lib")
                listdir = os.listdir(application_path)
                re_listdir = os.listdir(application_release_path)
                # 补充剩余空缺文件
                for x in re_listdir:
                    if x not in listdir:
                        shutil.copy(application_release_path + '/' + x, application_path + "/" + x)
            if os.path.isfile(application_path + "/start.sh"):
                os.chdir(application_path)
                os.chmod(application_path + "/start.sh", stat.S_IRWXU)
                os.system(application_path + "/start.sh > nohup.out 2>&1 &")


def main():
    allFileToDeployM()
    deployApplication()
    print("部署成功")


if __name__ == '__main__':
    main()

```