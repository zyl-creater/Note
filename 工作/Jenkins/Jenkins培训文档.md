# Jenkins 开发背景

## 1.1 实现版本输出流程规范化

包括打包材料输出和客制化内容及时提交；当前打包材料存在输出不规范的情况，主要是没有用autobuild编译，存在人为风险；同时客制化部分也存在不及时提交svn的情况，会有交接风险和客制化数据丢失风险；

## 1.2 减少版本编译耗时

当前软件输出流程由集成软件项目支撑和版本配置组同事负责，从autobuild材料到版本再配置流程较繁琐，存在时延。

# Jenkins 开发环境介绍

## 2.1 Linux服务器环境

### 运行环境

服务器 IP 地址：10.10.61.27，10.10.61.121

服务器登录地址：[http://10.10.61.27:8084/](http://10.10.61.27:8084/)

### 运行空间

在 Jenkins 服务中，所有的项目都集中在该服务器上，以 Orange 项目举例，一个完整的项目主要包括以下三个空间：

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/d5553cd0-8430-4246-a6ec-733c10042067.png)

1. 构建空间
    

存储项目编译的配置信息，编译 log 等；在27服务器上

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/30a6ed83-d3ea-4741-9d7b-446ca55148f5.png)

2. 打包空间
    

存储项目打包工程，脚本工具，编译属性等信息；在121服务器上

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/0795ebdf-31a1-49fd-93fe-4cf98805581d.png)

其中，jenkins pipeline 环境调用的本地脚本位于
![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/f3acc913-3b85-4a4d-bb6d-e415a77a4019.png)

再次，pipeline 环境要与pipeline_function.sh 进行交互和参数传递，受限于 pipeline Groovy 语法，需要中间文件进行参数传递，autobuild_info/package_info/properties 文件作为传递介质。后面对三个文件进行详细解释。

3. 代码空间
    

与 autobuild 代码空间路径一致，只存放 SDK 代码；在121服务器
![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/eac87a83-e9eb-4d77-9996-044c72b4b2ac.png)

Jenkins 环境中有多种项目类型：包括：freestyle 项目，maven 项目，pipeline 项目，External job 项目，MultiJob 项目，以及多分支流水线等；之前有用过 freestyle 项目，发现这种项目在软件配合打包过程不如 pipeline 有效，因此后续全部改为 pipeline 项目；

## 2.2 Pipeline 使用环境

Jenkins 环境中有多种项目类型：包括：freestyle 项目，maven 项目，pipeline 项目，External job 项目，MultiJob 项目，以及多分支流水线等；之前有用过 freestyle 项目，发现这种项目在软件配合打包过程不如 pipeline 有效，因此后续全部改为 pipeline 项目；

1. Jenkins Pipeline 介绍：
    

pipeline 是 Jenkins 的一个插件，Jenkins Pipeline 可以通过基于脚本的语言，如 Groovy 来定义整个软件拉取编译发布和测试；具有可编辑程度较高，多线程运行等优点；

2. 支持脚本兼容：
    

支持： Groovy，shell，python，ruby 等脚本；其中Groovy 是 Jenkins Pipeline 的默认脚本语言，是一种基于 Java 平台的面向对象编程语言，具有简洁、灵活和易于扩展的特点。在 Jenkins Pipeline 中，可以使用 Groovy 语言来定义流水线中的各个步骤和操作。

3. pipeline 项目结构：
    

流水线部分：该部分主要使用 Groovy 脚本进行编写，下面是一个典型的流水线结构：关键字包括：environment（定义环境变量），stages（主结构，每个 pipeline 项目只能有一个），parallel（该关键字下的 stage 可并行执行），stage（程序块），steps（程序块中的执行任务）等。

```
pipeline {
    agent any
    environment{
        SDMC_PIPELINE='true'
    }
    stages {
        parallel{
            stage('Autobuild') {
                steps {
                    sh 'echo Autobuild'
                }
            }
            stage('Custom_Package') {
                steps {
                    sh 'echo Custom_Package'
                }
            }
            stage('Collect_Build_Info') {
                steps {
                    sh 'echo Collect_Build_Info'
                }
            }
        }
    }
}
```

# Jenkins 主要模块

主要模块设计如下

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/02aa0d4b-67cb-4895-b690-e8d6a3b98e2d.png)

## 3.1 Jenkins Pipeline

Pipeline 在 SDMC 的编译模块中，定义了整个项目的运行流程和结构，是主干部分；如下图所示：

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/26b9d087-3055-4594-a8ee-5a018bccd6c2.png)

## 3.2 Jenkins Shell

Jenkins Shell 是 Pipeline 用于连接 SDMC 内部脚本的一个桥梁，在 pipeline 使用环境中，我们在 pipeline 中调用 shell 脚本，通过扩展 shell 脚本的方式，来提高脚本的兼容性；Jenkins Shell 使用sdk_patch/tool 的仓库，路径位于sdk_patch/tool/jenkins/pipeline_function.sh；如第一步中 download_sdk_tools ，更新所有git、svn仓库

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/67abeba2-06fa-4af8-a4b1-7712bae990a4.png)

## 3.3 Shell Script

Shell 脚本是SDMC的内部脚本，在shell脚本中实现了autobuild，打包环境的加载，对版本的号的处理，实现了状态集信息的收集，和一些特殊需求的实现

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/7a3388f3-5cce-4789-85ef-469afb7a4d17.png)

# 4. Pipeline和shell脚本分析

注意：因为jenkins 和 pipeline 的特殊性，部分参数是无法在shell脚本和pipeline的执行块之间直接传递，需要将参数写入文本中进行传递

脚本设计

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/57a9179d-b86c-4613-8be9-30f8cfe14041.png)

pipeline脚本

```
pipeline {
	agent any

	//声明环境变量
	environment {

		//PROPERTIES_FILE = "properties"
		SVN_PATH_NAME="${JENKINS_PROJECT_PACKAGE_PATH_LASTEST}/jenkins/${BUILD_ID}"
		SDK_TOOLS="${WORKSPACE}/sdk_tool"
		JENKINS_PIPELINE_SCRIPT = "${WORKSPACE}/sdk_tool/jenkins/pipeline_function.sh"

		//################ 下面一般不需要修改 ####################
		PACK_WORK_DIR="${WORKSPACE}/${PACK_WORK_RELATIVE_PATH}"

		//打包工程仓库
		PACKAGE_TOOLS_GIT_REPOSITORY="${PACKAGE_TOOLS_GIT_REPOSITORY_PROP}"

		PACKAGE_TOOLS="${PACK_WORK_DIR}/tools"
		PACK_OUT_DIR="${PACK_WORK_DIR}/out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}"

		PLATFORM_DIR="${PACK_WORK_DIR}/platform"
		PLATFORM_PROJECT_DIR="${PLATFORM_DIR}/${PACK_PARAS_PLATFORM}"
		PROJECT_DIR="${PLATFORM_PROJECT_DIR}/${PACK_PARAS_PROJECT}"
		PUBLIC_DIR="${PLATFORM_DIR}/platform_public"

		//# 获取git svn id和fingerprint等信息，保存在${BUILD_SELF_PROPERTIES_FILE} 中
		PROPERTIES_FILE_NAME="properties"
		BUILD_SELF_PROPERTIES_FILE="${WORKSPACE}/${PROPERTIES_FILE_NAME}"

		//编译后信息收集：
		AUTOBUILD_INFO="autobuild_info"
		INFO_FILE="package_info"
		AUTOBUILD_INFO_COLLECT_FILE="${WORKSPACE}/${AUTOBUILD_INFO}"
		PACKAGE_INFO_COLLECT_FILE="${WORKSPACE}/${INFO_FILE}"
		SVN_PUSH_DIR_URL="${SVN_PUSH_DIR_URL}"

		SVN_MATERIALS_UPLOAD_PATH="${SVN_MATERIALS_UPLOAD_PATH}"
		JENKINS_SDK_MANIFEST_SH=''

		PACK_PROJ="pack_proj"
		PACK_PROJ_COLLECT_FILE="${WORKSPACE}/${PACK_PROJ}"
	}
	stages{
		//更新所有git、svn仓库
		stage('download_sdk_tools'){
			//更新sdk_tools
			steps {
			sh """
				if [ -d ${SDK_TOOLS} ];then
					cd ${SDK_TOOLS}
					git checkout .
					git pull
				else
					mkdir ${SDK_TOOLS} -p
					git clone amlogic_l_group@10.10.61.21:/home/svn/sdmc_lib/sdk_patch_tool/tool.git sdk_tool --depth 1
				fi
			"""
			}
		}
		stage('download_config'){
			parallel{
				stage('download_custom_project'){
					//更新客制化内容
					steps {
						sh """
						source ${JENKINS_PIPELINE_SCRIPT};
						download_custom_project;
						"""
					}
				}
				stage('download_public_project'){
					//更新打包public内容
					steps {
						sh """
						source ${JENKINS_PIPELINE_SCRIPT};
						download_public_project;
						"""
					}
				}
				stage('download_package_tools'){
					//更新打包工程
					steps {
						sh """
						source ${JENKINS_PIPELINE_SCRIPT};
						download_package_tools;
						"""
					}
				}
			}
		}

		//解析patch_list
		stage('parse_patch_list'){
			steps {
				script {
					def JENKINS_PROJECT_NAME,ROOT_DIR_SH
					def cause = "${currentBuild.getBuildCauses()[0].shortDescription}"
					sh """
						echo patch list:\${PATCH_LIST}
						cd ${WORKSPACE}
						if [ ! -d "${PROPERTIES_FILE_NAME}" ];then
							touch ${PROPERTIES_FILE_NAME}
							touch ${AUTOBUILD_INFO}
							touch ${INFO_FILE}
							touch ${PACK_PROJ}
						fi

						cd ${SDK_TOOLS}/auto_build/projects/android_s/

						#s905x2
						JENKINS_SOC_VISION=`find . -name ${PATCH_LIST}.sh | cut -d '/' -f 2`
						cd ${SDK_TOOLS}/auto_build/projects/android_s/\${JENKINS_SOC_VISION}

						ROOT_DIR_SH=`cat  ${PATCH_LIST}.sh  | sed -n '6p' |tr '\t' ' ' | cut -d '~' -f 2 | cut -d ' ' -f 1`

						#s905x2_sdk0908_orange
						JENKINS_PROJECT_NAME=`cat  ${PATCH_LIST}.sh  | sed -n '7p' |tr '\t' ' ' | cut -d '=' -f 2 | cut -d ' ' -f 1`

						#openlinux/ott/s-amlogic-q-vendor-20220908-gtvs.xml
						JENKINS_SDK_MANIFEST=`strings ${PATCH_LIST}.sh | grep 'SDK_MANIFEST' | sed '/\${/d' | tr '\t' ' ' | cut -d '=' -f 2 | cut -d ' ' -f 1`

						#franklink
						JENKINS_SDK_PLATFORM=`cat  ${PATCH_LIST}.sh | sed -n '8p' |tr '\t' ' ' | cut -d '=' -f 2 | cut -d ' ' -f 1`

						#清空package_info
						echo " " > ${PACKAGE_INFO_COLLECT_FILE}

						echo ROOT_DIR=\${ROOT_DIR_SH} > ${BUILD_SELF_PROPERTIES_FILE};
						echo "ROOT_DIR='\${ROOT_DIR_SH}'" > ${AUTOBUILD_INFO_COLLECT_FILE};

						echo JENKINS_PROJECT_NAME=\${JENKINS_PROJECT_NAME} >> ${BUILD_SELF_PROPERTIES_FILE}
						echo "JENKINS_PROJECT_NAME='\${JENKINS_PROJECT_NAME}'" >> ${AUTOBUILD_INFO_COLLECT_FILE}

						echo JENKINS_SDK_PLATFORM=\${JENKINS_SDK_PLATFORM} >> ${BUILD_SELF_PROPERTIES_FILE}
						echo "JENKINS_SDK_PLATFORM='\${JENKINS_SDK_PLATFORM}'" >> ${AUTOBUILD_INFO_COLLECT_FILE}

						echo JENKINS_SDK_MANIFEST=\${JENKINS_SDK_MANIFEST} >> ${BUILD_SELF_PROPERTIES_FILE}
						echo "JENKINS_SDK_MANIFEST='\${JENKINS_SDK_MANIFEST}'" >> ${AUTOBUILD_INFO_COLLECT_FILE}

						#获取上一次打包材料路径
						USER=`cat  ${PACK_PROJ_COLLECT_FILE}  | grep 'user='|awk -F '=' '{print \$2}'`
						USERDEBUG=`cat  ${PACK_PROJ_COLLECT_FILE}  | grep 'userdebug='|awk -F '=' '{print \$2}'`
						cat ${PACK_PROJ_COLLECT_FILE}
						if [ -n "$PACKAGE_PATH_USER" ]; then
							echo NEW_PACKAGE_PATH_USER=\${PACKAGE_PATH_USER} >> ${BUILD_SELF_PROPERTIES_FILE}
						else
							echo NEW_PACKAGE_PATH_USER=\${USER} >> ${BUILD_SELF_PROPERTIES_FILE}
						fi
						if [ -n "$PACKAGE_PATH_USERDEBUG" ]; then
							echo NEW_PACKAGE_PATH_USERDEBUG=\${PACKAGE_PATH_USERDEBUG} >> ${BUILD_SELF_PROPERTIES_FILE}
						else
							echo NEW_PACKAGE_PATH_USERDEBUG=\${USERDEBUG} >> ${BUILD_SELF_PROPERTIES_FILE}
						fi

						LOCAL_DATE=`date +%Y-%m-%d-%H%M%S`
						echo LOCAL_DATE=\${LOCAL_DATE} >> ${BUILD_SELF_PROPERTIES_FILE}

						#获取构建者名字和构建状态
						if echo "$cause" | grep -q "time"; then
						echo BUILD_TYPE=both >> ${BUILD_SELF_PROPERTIES_FILE}
						echo "BUILD_PERSONNEL='Jenkins'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
						echo "BUILD_METHOD='定时构建'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
						else
						echo BUILD_TYPE=\${CHOSE_BUILD_TYPE} >> ${BUILD_SELF_PROPERTIES_FILE}
						BUILD_PERSONNEL=`echo "$cause"  |cut -d ' ' -f 4`
						echo "BUILD_PERSONNEL='\${BUILD_PERSONNEL}'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
						echo "BUILD_METHOD='手动构建'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
						fi
						echo "$BUILD_VERSION"
					"""

					def props = readProperties file: "${PROPERTIES_FILE_NAME}"
					BUILD_TYPE="${props['BUILD_TYPE']}"
					ROOT_DIR="${props['ROOT_DIR']}"
					JENKINS_PROJECT_NAME="${props['JENKINS_PROJECT_NAME']}"
					JENKINS_SDK_MANIFEST="${props['JENKINS_SDK_MANIFEST']}"

					//franklin
					JENKINS_SDK_PLATFORM="${props['JENKINS_SDK_PLATFORM']}"
					//sdk autobuild path
					JENKINS_AUTOBUILD_SDK="/home/jenkins${ROOT_DIR}/${JENKINS_PROJECT_NAME}_${BUILD_TYPE}"
					JENKINS_AUTOBUILD_SDK_PATH="/home/jenkins${ROOT_DIR}/${JENKINS_PROJECT_NAME}_${BUILD_TYPE}/${JENKINS_SDK_MANIFEST}"
					JENKINS_PACKAGE_NAME='${JENKINS_SOC_VISION}_master-${JENKINS_SDK_VISION}'
				}
			}
		}

		//清理out缓存
		stage('clear_out_cache'){
			when {
				expression {SDK_BUILD == 'true'}
			}
			steps {
				script {
					def props = readProperties file: "${PROPERTIES_FILE_NAME}"
					ROOT_DIR="${props['ROOT_DIR']}"
					JENKINS_PROJECT_NAME="${props['JENKINS_PROJECT_NAME']}"
					JENKINS_SDK_MANIFEST="${props['JENKINS_SDK_MANIFEST']}"
					sh """
					source ${JENKINS_PIPELINE_SCRIPT};
					clear_out_cache ${ROOT_DIR} ${JENKINS_PROJECT_NAME} ${JENKINS_SDK_MANIFEST} ${BUILD_TYPE};
					"""
				}
				
			}
		}

		//开始autobuild
		stage('start_autobuild'){
			failFast true
			when {
				expression {SDK_BUILD == 'true'}
			}
			parallel{
				stage('truely_autobuild') {
					steps {
						script {
						def cause = "${currentBuild.getBuildCauses()[0].shortDescription}"
						sh """
						echo ${cause}
						source ${JENKINS_PIPELINE_SCRIPT};
						init_auto_build_env;
						start_auto_build $JENKINS_AUTOBUILD_SDK_PATH ${BUILD_TYPE};
						"""
						}
					}
				}
				stage('package_user') {
					when {
						expression {BUILD_TYPE == 'both'}
					}
					steps {
						script {
							while (true) {
								def props = readProperties file: "${BUILD_SELF_PROPERTIES_FILE}"
								USER_BUILD_DONE="${props['USER_BUILD_DONE']}"
								if (USER_BUILD_DONE == '1') {
									// user build done
									sh """
									source ${JENKINS_PIPELINE_SCRIPT};
									#推送user材料到svn
									push_materials_to_svn ${JENKINS_AUTOBUILD_SDK_PATH} ${SVN_MATERIALS_UPLOAD_PATH} user ${BUILD_TYPE};
									"""
									break
								} else {
									sh """
									source ${JENKINS_PIPELINE_SCRIPT};
									#check 材料是否生成
									check_user_materials_build_done "${JENKINS_AUTOBUILD_SDK_PATH}" $JENKINS_SDK_PLATFORM;
									"""
									echo "waiting for user build done..."
									sleep 300
								}
							}
							def props = readProperties file: "${BUILD_SELF_PROPERTIES_FILE}"
							SVN_OTAPACKAGE_LINK_user="${props['SVN_OTAPACKAGE_LINK_user']}"
							NEW_PACKAGE_PATH_USERDEBUG="${props['NEW_PACKAGE_PATH_USERDEBUG']}"
							LOCAL_DATE="${props['LOCAL_DATE']}"
							sh """
							echo "user=$SVN_OTAPACKAGE_LINK_user" > ${PACK_PROJ_COLLECT_FILE}
							source ${JENKINS_PIPELINE_SCRIPT};
							start_package user ${SVN_OTAPACKAGE_LINK_user} ${BUILD_TYPE} ${LOCAL_DATE};
							"""
						}
					}
				}
			}
		}

		//打包user版本
		stage('package_user_except_both'){
			when {
				allOf{
					expression {SDK_BUILD == 'true'};
					expression {BUILD_TYPE == 'user'}
				}
			}
			steps {
				script {
					sh """
						source ${JENKINS_PIPELINE_SCRIPT};
						#推送user材料到svn
						#放开
						push_materials_to_svn ${JENKINS_AUTOBUILD_SDK_PATH} ${SVN_MATERIALS_UPLOAD_PATH} user ${BUILD_TYPE};
					"""
					def props = readProperties file: "${BUILD_SELF_PROPERTIES_FILE}"
					SVN_OTAPACKAGE_LINK_user="${props['SVN_OTAPACKAGE_LINK_user']}"
					NEW_PACKAGE_PATH_USERDEBUG="${props['NEW_PACKAGE_PATH_USERDEBUG']}"
					LOCAL_DATE="${props['LOCAL_DATE']}"
					sh """
						echo "user=$SVN_OTAPACKAGE_LINK_user" > ${PACK_PROJ_COLLECT_FILE}
						echo "userdebug=$NEW_PACKAGE_PATH_USERDEBUG" >> ${PACK_PROJ_COLLECT_FILE}
						source ${JENKINS_PIPELINE_SCRIPT};
						start_package user ${SVN_OTAPACKAGE_LINK_user} ${BUILD_TYPE} ${LOCAL_DATE};
					"""
				}
			}
		}

		//打包debug版本
		stage('package_debug'){
			when {
				allOf{
					expression {SDK_BUILD == 'true'};
					expression {BUILD_TYPE == 'both' || BUILD_TYPE == 'userdebug'}
				}
			}
			steps {
				script {
					sh """
						source ${JENKINS_PIPELINE_SCRIPT};
						#推送debug材料到svn
						#放开
						push_materials_to_svn ${JENKINS_AUTOBUILD_SDK_PATH} ${SVN_MATERIALS_UPLOAD_PATH} userdebug ${BUILD_TYPE};
					"""
					def props = readProperties file: "${BUILD_SELF_PROPERTIES_FILE}"
					SVN_OTAPACKAGE_LINK_userdebug="${props['SVN_OTAPACKAGE_LINK_userdebug']}"
					NEW_PACKAGE_PATH_USER="${props['NEW_PACKAGE_PATH_USER']}"
					LOCAL_DATE="${props['LOCAL_DATE']}"
					sh """
						if [ "$BUILD_TYPE" == "both" ]; then
							echo "userdebug=$SVN_OTAPACKAGE_LINK_userdebug" >> ${PACK_PROJ_COLLECT_FILE}
						else
							echo "user=$NEW_PACKAGE_PATH_USER" > ${PACK_PROJ_COLLECT_FILE}
							echo "userdebug=$SVN_OTAPACKAGE_LINK_userdebug" >> ${PACK_PROJ_COLLECT_FILE}
						fi
						source ${JENKINS_PIPELINE_SCRIPT};
						start_package userdebug ${SVN_OTAPACKAGE_LINK_userdebug} ${BUILD_TYPE} ${LOCAL_DATE};
					"""
				}
				
			}
		}

		//只打包
		stage('only_package'){
			when {
				expression {SDK_BUILD == 'false'};
			}
			steps {
				script {
					def props = readProperties file: "${BUILD_SELF_PROPERTIES_FILE}"
					NEW_PACKAGE_PATH_USER="${props['NEW_PACKAGE_PATH_USER']}"
					NEW_PACKAGE_PATH_USERDEBUG="${props['NEW_PACKAGE_PATH_USERDEBUG']}"
					LOCAL_DATE="${props['LOCAL_DATE']}"
					if ( SELECT_PACKAGE == "custom" ) {
						sh """
						source ${JENKINS_PIPELINE_SCRIPT};
						start_package custom ${CUSTOM_PACKAGE_PATH} ${BUILD_TYPE} ${LOCAL_DATE};
						"""
					}
					if ( SELECT_PACKAGE == "user" ) {
						sh """
						echo "user=$NEW_PACKAGE_PATH_USER" > ${PACK_PROJ_COLLECT_FILE}
						echo "userdebug=$NEW_PACKAGE_PATH_USERDEBUG" >> ${PACK_PROJ_COLLECT_FILE}
						source ${JENKINS_PIPELINE_SCRIPT};
						start_package user ${NEW_PACKAGE_PATH_USER} ${BUILD_TYPE} ${LOCAL_DATE};
						"""
					}
					if ( SELECT_PACKAGE == "userdebug" ) {
						sh """
						echo "user=$NEW_PACKAGE_PATH_USER" > ${PACK_PROJ_COLLECT_FILE}
						echo "userdebug=$NEW_PACKAGE_PATH_USERDEBUG" >> ${PACK_PROJ_COLLECT_FILE}
						source ${JENKINS_PIPELINE_SCRIPT};
						start_package userdebug ${NEW_PACKAGE_PATH_USERDEBUG} ${BUILD_TYPE} ${LOCAL_DATE};
						"""
					}
				}
			}
		}

		//上传正式版本到svn
		stage('push_image_to_svn'){
			steps {
				script {
					def props = readProperties file: "${BUILD_SELF_PROPERTIES_FILE}"
					LOCAL_IMAGE_DIR="${props['LOCAL_IMAGE_DIR']}"
					sh """
					source ${JENKINS_PIPELINE_SCRIPT};
					push_image_to_svn $LOCAL_IMAGE_DIR;
					"""
				}
			}
		}

		//上传base包到指定目录
		stage('push_base_zip_to_server'){
			when {
				expression {SELECT_PUSH_CUSTOM == 'true'};
			}
			steps {
				script {
					def props = readProperties file: "${BUILD_SELF_PROPERTIES_FILE}"
					LOCAL_IMAGE_DIR="${props['LOCAL_IMAGE_DIR']}"
					FINAL_NAME_USER="${props['FINAL_NAME_USER']}"
					FINAL_NAME_USERDEBNUG="${props['FINAL_NAME_USERDEBNUG']}"
					sh """
					source ${JENKINS_PIPELINE_SCRIPT};
					upload_base_zip_server $LOCAL_IMAGE_DIR $FINAL_NAME_USER $FINAL_NAME_USERDEBNUG;
					"""
				}
			}
		}

		//收集、整理构建信息
		stage('collect_package_info'){
			steps {
				//def props = readProperties file: "${BUILD_SELF_PROPERTIES_FILE}"
				//SVN_OTAPACKAGE_LINK="${props['SVN_OTAPACKAGE_LINK']}"
				sh """
				source ${JENKINS_PIPELINE_SCRIPT};
				collect_package_info $JENKINS_AUTOBUILD_SDK_PATH $JENKINS_AUTOBUILD_SDK ${BUILD_TYPE};
				"""
			}
		}
	}

	//编译后操作
	post{
		always {
			sh "printenv | sort"
		}
		success {
			script {
				def props = readProperties file: '${PROPERTIES_FILE_NAME}'
				PATCH_TOOL_COMMIT_INFO="${props['PATCH_TOOL_COMMIT_INFO']}"
				load "${AUTOBUILD_INFO_COLLECT_FILE}"
				load "${PACKAGE_INFO_COLLECT_FILE}"
				def fileContent = readFile "$PACK_WORK_RELATIVE_PATH/out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}/${LOCAL_IMAGE_DIR}/repeat_apk.txt"
					buildDescription "<ul>"+
						"<li>***************** 构建信息 ******************</li>"+
						"<li>构建人员 ： ${BUILD_PERSONNEL}</li>"+
						"<li>构建时间 ： ${BUILD_TIMESTAMP}</li>"+
						"<li>构建日志 ： <a href='${BUILD_URL}console'>${BUILD_URL}console</a></li>"+
						"<li>工作目录 ： <a href='${BUILD_URL}execution/node/3/ws'>${BUILD_URL}execution/node/3/ws</a></li>"+
						"<li>固件下载路径 ： <a href='${BUILD_URL}execution/node/3/ws/${PACK_WORK_RELATIVE_PATH}/out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}/${LOCAL_IMAGE_DIR}'> 固件输出路径 </a></li>"+
						"<li>***************** 材料信息 ******************</li>"+
						"<li>是否使用SDK_BUILD ： ${SDK_BUILD}</li>"+
						"<li>SDK manifast ： ${JENKINS_SDK_MANIFEST}</li>"+
						"<li>PATCH_LIST ： ${PATCH_LIST}</li>"+
						"<li>编译平台 ： ${JENKINS_SDK_PLATFORM}</li>"+
						"<li>编译类型 ： ${BUILD_TYPE}</li>"+
						"<li>amlogic_s_stb最新分支：  ${PATCH_BRANCH}</li>"+
						"<li>amlogic_s_stb最新提交：  ${PATCH_TOOL_COMMIT_INFO}</li>"+
						"<li>安全补丁 ： ${SECURITY_PATCH}</li>"+
						"<li>GTVS版本： ${GTV_VERSION}</li>"+
						"<li>强制PATCH版本： ${MANDATORY_PATCH}</li>"+
						"<li>***************** 版本配置 ******************</li>"+
						"<li>项目机型 ： ${JOB_NAME}</li>"+
						"<li>指纹信息 ： ${FINGERPRINT}</li>"+
						"<li>版本号 ： ${BUILD_VER}</li>"+
						"<li>打包环境 svn 最新提交 ： ${PACKAGE_CUSTOM_COMMIT_INFO}</li>"+
						"<li>package tool 最新提交 ： ${PACKAGE_TOOL_COMMIT_INFO}</li>"+
						"<li>打包材料[user] ： ${PACKAGE_SVN_OTAPACKAGE_LINK_user}</li>"+
						"<li>打包材料[userdebug]  ： ${PACKAGE_SVN_OTAPACKAGE_LINK_userdebug}</li>"+
						"<li>打包材料[custom]  ： ${PACKAGE_SVN_OTAPACKAGE_LINK_custom}</li>"+
						"<li>正式版本上传路径  ： ${SVN_IMAGE_UPLOAD_DIR_PATH}</li>"+
						"<li>重复APK检查  ： ${REPEAT_APK}</li>"+
						"<li><pre style='color: #ef0d0d!important;'>${fileContent}</pre></li>"+
						"</ul>"
			}
		}
		failure {
			script {
				mail to: 'harry_zhu@sdmctech.com',
				subject: "Running Pipeline: ${currentBuild.fullDisplayName}",
				body: " ${JOB_NAME} -Build # ${BUILD_NUMBER} - error!\n Check console output at ${BUILD_URL} to view the results."
			}
		}
	}
}
```

SDMC内部脚本

```
#!/bin/bash

#下载项目配置
function download_custom_project()
{
	echo "download_custom_project start"
	if [ -d ${PLATFORM_PROJECT_DIR} ];then
		rm -rf ${PLATFORM_PROJECT_DIR}
	fi
	mkdir -p ${PLATFORM_PROJECT_DIR}
	cd ${PLATFORM_PROJECT_DIR}
	svn co ${PROJECT_URL} --username ${SVN_USERNAME} --password ${SVN_PASSWORD}
	echo "download_custom_project end"
}

#下载public
function download_public_project()
{
	if [ -d $PUBLIC_DIR ];then
		rm -rf $PUBLIC_DIR
	fi
	mkdir -p ${PLATFORM_DIR}
	cd ${PLATFORM_DIR}
	svn co ${PUBLIC_URL} --username ${SVN_USERNAME} --password ${SVN_PASSWORD}
}

#下载打包工程
function download_package_tools()
{
	if [ -d ${PACKAGE_TOOLS} ];then
		cd ${PACKAGE_TOOLS}
		git checkout .
		git pull
	else
		mkdir ${PACK_WORK_DIR} -p
		cd ${PACK_WORK_DIR}
		git clone ${PACKAGE_TOOLS_GIT_REPOSITORY} --depth 1
		ln -s tools/Makefile .
	fi
}

#清理out缓存--待优化
function clear_out_cache()
{
	BUILD_TYPE=$4
	PATH=/home/jenkins/"$1"/"$2_${BUILD_TYPE}"/$3
	if [ -d ${PATH} ];then
		clear_out_cache_true "${PATH}"
	fi
}

#实际清理out缓存
function clear_out_cache_true()
{
	init_auto_build_env
	cd $1
	DATE=`/bin/date +%Y-%m-%d-%H%M%S`
	CACHE_DIR=cache-${DATE}

	mkdir ${CACHE_DIR}

	MASTER=$(ls | grep "master" | sed -n '1p')
	len=${#MASTER}
	if [ $len -gt 0 ]; then
		echo "MASTER len > 0"
		mv *master* ${CACHE_DIR}
	else
		echo "MASTER len = 0"
	fi

	OUT_DEBUG=$(ls | grep "out" | sed -n '1p')
	len=${#OUT_DEBUG}
	if [ $len -gt 0 ]; then
		echo "OUT1 len > 0"
		mv *out* ${CACHE_DIR}
	else
		echo "OUT1 len = 0"
	fi

	OUT_USER=$(ls ../ | grep "out" | sed -n '1p')
	len=${#OUT_USER}
	if [ $len -gt 0 ]; then
		echo "OUT2 len > 0"
		mv ../*out* ${CACHE_DIR}
	else
		echo "OUT2 len = 0"
	fi

	rm -rf cache*
}

#配置autobuild环境
function init_auto_build_env()
{
	echo "init_auto_build_env start"
	#export BUILD_AUTHOR=$BUILD_AUTHOR
	export PATCH_LIST=$PATCH_LIST
	export BUILD_COVER=$BUILD_COVER
	export PATH=$PATH:/home/jenkins/bin:/home/jenkins/.local/bin:/opt/jdk-11/bin//bin:/opt/gnutools/arc2.3-p0/elf32-4.2.1/bin:/opt/gnutools/arc2.3-p0/uclibc-4.2.1/bin:/opt/arc-4.8-amlogic-20130904-r2/bin:/opt/CodeSourcery/Sourcery_G++_Lite/bin:/opt/CodeSourcery/Sourcery_G++_Lite/arm-none-eabi/bin:/opt/CodeSourcery/Sourcery_G++_Lite/arm-none-linux-gnueabi/bin:/opt/gcc-linaro-arm-linux-gnueabihf-4.9-2014.05_linux/bin:/opt/gcc-linaro-aarch64-linux-gnu-4.9-2014.09_linux/bin:/opt/gcc-linaro-aarch64-none-elf-4.8-2013.11_linux/bin:/opt/gcc-linaro-6.3.1-2017.02-x86_64_arm-linux-gnueabihf/bin:/opt/riscv-none-gcc/7.2.0-4-20180606-1631/bin:/opt/gcc-linaro-6.3.1-2017.02-x86_64_arm-linux-gnueabihf/bin:/opt/riscv-none-gcc/7.2.0-4-20180606-1631/bin:/opt/gcc-linaro-aarch64-linux-gnu-4.9-2014.09_linux/bin:/bin
	echo "init_auto_build_env end"
}

#引用auto_build脚本
function start_auto_build()
{
	#$1 
	#$2 ${cause}
	echo "start_auto_build start"
	set -e

	echo "${2}"
	if [ "$ROLL_BACK" == "true" ];then
		cd $WORKSPACE
		svn export --force ${ROOL_BACK_FILE_PATH} --username ${SVN_USERNAME} --password ${SVN_PASSWORD}
		ROOL_BACK_FILE=`echo ${ROOL_BACK_FILE_PATH} |awk -F '/' '{print $NF}'`
		$WORKSPACE/sdk_tool/auto_build/auto_build_s.sh $PATCH_LIST $2 $BUILD_COVER $DEBUG_PATCH_LIST ${WORKSPACE}/$ROOL_BACK_FILE
	else
		$WORKSPACE/sdk_tool/auto_build/auto_build_s.sh $PATCH_LIST $2 $BUILD_COVER $DEBUG_PATCH_LIST none
	fi
	if [ $? -eq 0 ];then 
		cd $1
		FINISH_BUILD=$(ls | grep ${JENKINS_PACKAGE_NAME}_user)
		if [ ! -n "$FINISH_BUILD" ];then
			echo "FINISH_BUILD=1" >> ${BUILD_SELF_PROPERTIES_FILE}
			exit 1
		fi
	fi
	echo "start_auto_build end"
}

#确认user材料是否编译出来
function check_user_materials_build_done()
{
	# $1 SDK 材料所在路径
	# $2 编译平台,如franklin

	if [ -f $1/../out_user/target/product/$2/aml_upgrade_package.img ];then
		echo "allen debug user_build_done_truly"
		echo "USER_BUILD_DONE=1" >> ${BUILD_SELF_PROPERTIES_FILE}
	fi
}

#确认材料是否编译出来
function push_materials_to_svn()
{
	# $1 SDK 材料所在路径
	# $2 SVN 材料上传路径
	# $3 USER OR DEBUG
	echo "push_materials_to_svn start"
	BUILD_TYPE=$4
	SVN_MATERIALS_UPLOAD_PATH=$2
	TMP_TYPE=$3

	cd $1
	USER_MATERIALS_NAME=$(ls | grep "_${TMP_TYPE}_" | sed -n '1p')
	len=${#USER_MATERIALS_NAME}
	if [ $len -gt 0 ]; then
		echo "SVN_OTAPACKAGE_LINK_${TMP_TYPE}='${SVN_MATERIALS_UPLOAD_PATH}/${USER_MATERIALS_NAME}'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
		#echo SVN_OTAPACKAGE_LINK_${TMP_TYPE}=${SVN_MATERIALS_UPLOAD_PATH}/${USER_MATERIALS_NAME} >> ${PROPERTIES_FILE_NAME}
		echo SVN_OTAPACKAGE_LINK_${TMP_TYPE}=${SVN_MATERIALS_UPLOAD_PATH}/${USER_MATERIALS_NAME} >> ${BUILD_SELF_PROPERTIES_FILE}
		push_to_svn $1 ${USER_MATERIALS_NAME} ${SVN_MATERIALS_UPLOAD_PATH}
	else
		echo "USER_MATERIALS_NAME len = 0"
		exit 1 #出错，返回
	fi
	if [ "$BUILD_TYPE" == "user" ]; then
		echo "SVN_OTAPACKAGE_LINK_userdebug='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	fi
	if [ "$BUILD_TYPE" == "userdebug" ]; then
		echo "SVN_OTAPACKAGE_LINK_user='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	fi
	echo "push_materials_to_svn end"
}

#推送材料到svn 
function push_to_svn()
{
	# $1 需要上传的文件夹 路径
	# $2 需要上传的文件夹
	# $3 需要推送的svn路径
	echo "push_to_svn start"
	cd $1
	#DATE=`date +%Y%m%d%H%M%S`
	svn mkdir $3/$2 --username ${SVN_USERNAME} --password ${SVN_PASSWORD} -m "jenkins autobuild mkdir materials svn upload path"
	svn import $2 $3/$2 --username ${SVN_USERNAME} --password ${SVN_PASSWORD} -m "jenkins autobuild upload materials"
	echo "push_to_svn end"
}

#配置打包环境变量
function init_packaging_env()
{
	echo "init_packaging_env start"
	cd ${PACK_WORK_DIR}
	export JAVA_HOME=/opt/jdk-11
	export JRE_HOME=${JAVA_HOME}
	export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
	export PATH=./tools/android-utils/host/bin:${JAVA_HOME}/bin:$PATH
	echo ${JENKINS_PROJECT_PACKAGE_PATH}
	echo "init_packaging_env end"

}

#开始打包
function start_package()
{
	#$1 编译类型
	#$2 打包材料svn路径宏定义
	#$3 BUILD_TYPE
	#$4 LOCAL_DATE
	TMP_TYPE=$1
	BUILD_TYPE=$3
	export SVN_OTAPACKAGE_LINK=$2

	cd ${PACK_WORK_DIR}

	init_packaging_env

	if [ "$TMP_TYPE" == "custom" ]; then
		if echo "$SVN_OTAPACKAGE_LINK" | grep -q "userdebug"; then
			TMP_VER="userdebug"
		else
			TMP_VER="user"
		fi
	else
		TMP_VER="${TMP_TYPE}"
	fi
	if [ ${USE_NEW_VISION} == "true" ]; then
		newversion=${BUILD_ID}A
		VISION_DIR=${newversion}
		if [ "$TMP_VER" == "userdebug" ]; then
			PACK_PARAS_VERSION="debug$newversion"
		else
			PACK_PARAS_VERSION="$newversion"
		fi
		if [ "$TMP_VER" == "userdebug" ] && [ "$BUILD_TYPE" == "both" ]; then
			echo "NEW_BUILD_VERSION=${PACK_PARAS_VERSION}"
		else
			echo "NEW_BUILD_VERSION='$newversion'" >> ${PACKAGE_INFO_COLLECT_FILE}
			echo "NEW_BUILD_VERSION=$newversion" >> ${BUILD_SELF_PROPERTIES_FILE}
		fi
	else
		if [ -z "$BUILD_VERSION" ]; then
			exit 1
		fi
		VISION_DIR=${BUILD_VERSION}
		if [ "$TMP_VER" == "userdebug" ] && [ "$BUILD_TYPE" == "both" ]; then
			PACK_PARAS_VERSION=${BUILD_VERSION}.1
			echo "NEW_BUILD_VERSION=${PACK_PARAS_VERSION}"
		else
			PACK_PARAS_VERSION=${BUILD_VERSION}
			echo "NEW_BUILD_VERSION='$BUILD_VERSION'" >> ${PACKAGE_INFO_COLLECT_FILE}
			echo "NEW_BUILD_VERSION=$BUILD_VERSION" >> ${BUILD_SELF_PROPERTIES_FILE}
		fi
	fi

	LOCAL_IMAGE_DIR="${VISION_DIR}-$4"
	echo "LOCAL_IMAGE_DIR=$LOCAL_IMAGE_DIR" >>${BUILD_SELF_PROPERTIES_FILE}

	#####版本配置信息

	case ${TMP_TYPE} in
	"user")
		if [ "$BUILD_TYPE" == "both" ]; then
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_user='$SVN_OTAPACKAGE_LINK'" >> ${PACKAGE_INFO_COLLECT_FILE}
		else
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_user='$SVN_OTAPACKAGE_LINK'" >> ${PACKAGE_INFO_COLLECT_FILE}
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_userdebug='NA'" >> ${PACKAGE_INFO_COLLECT_FILE}
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_custom='NA'" >> ${PACKAGE_INFO_COLLECT_FILE}
		fi
		;;
	"userdebug")
		if [ "$BUILD_TYPE" == "both" ]; then
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_userdebug='$SVN_OTAPACKAGE_LINK'" >> ${PACKAGE_INFO_COLLECT_FILE}
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_custom='NA'" >> ${PACKAGE_INFO_COLLECT_FILE}
		else
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_user='NA'" >> ${PACKAGE_INFO_COLLECT_FILE}
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_userdebug='$SVN_OTAPACKAGE_LINK'" >> ${PACKAGE_INFO_COLLECT_FILE}
			echo "PACKAGE_SVN_OTAPACKAGE_LINK_custom='NA'" >> ${PACKAGE_INFO_COLLECT_FILE}
		fi
		;;
	"custom")
		echo "PACKAGE_SVN_OTAPACKAGE_LINK_user='NA'" >> ${PACKAGE_INFO_COLLECT_FILE}
		echo "PACKAGE_SVN_OTAPACKAGE_LINK_userdebug='NA'" >> ${PACKAGE_INFO_COLLECT_FILE}
		echo "PACKAGE_SVN_OTAPACKAGE_LINK_custom='$SVN_OTAPACKAGE_LINK'" >> ${PACKAGE_INFO_COLLECT_FILE}
		;;
	*)
		echo "TMP_TYPE error"
		;;
	esac
	if [ "$SDK_BUILD" == "false" ]; then
		echo "SVN_OTAPACKAGE_LINK_user='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
		echo "SVN_OTAPACKAGE_LINK_userdebug='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	fi

	####开始打包
	echo "package $TMP_TYPE done!!!!"
	make c=${PACK_PARAS_PROJECT} p=${PACK_PARAS_PLATFORM} t=${PACK_PARAS_T} k=${PACK_PARAS_KEY} v=${PACK_PARAS_VERSION}

	if [ "$TMP_VER" == "user" ]; then
		FINAL_NAME_USER=`cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH} && cat cache/package_name_string.txt`
		len=${#FINAL_NAME_USER}
		if [ $len -le 0 ]; then
			exit 1
		fi
		echo "FINAL_NAME_USER=$FINAL_NAME_USER" >>${BUILD_SELF_PROPERTIES_FILE}
		if [ "$BUILD_TYPE" != "both" ]; then
			echo "FINAL_NAME_USERDEBNUG=NA" >>${BUILD_SELF_PROPERTIES_FILE}
		fi
	fi
	if [ "$TMP_VER" == "userdebug" ]; then
		FINAL_NAME_USERDEBNUG=`cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH} && cat cache/package_name_string.txt`
		len=${#FINAL_NAME_USERDEBNUG}
		if [ $len -le 0 ]; then
			exit 1
		fi
		echo "FINAL_NAME_USERDEBNUG=$FINAL_NAME_USERDEBNUG" >>${BUILD_SELF_PROPERTIES_FILE}
		if [ "$BUILD_TYPE" != "both" ]; then
			echo "FINAL_NAME_USER=NA" >>${BUILD_SELF_PROPERTIES_FILE}
		fi
	fi

	FINAL_NAME=`cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH} && cat cache/package_name_string.txt`
	cd ${PACK_WORK_DIR}/out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}/
	if [ ! -d "${LOCAL_IMAGE_DIR}" ]; then
		mkdir -p ${LOCAL_IMAGE_DIR}
	fi
	mv *${FINAL_NAME}*.img ${LOCAL_IMAGE_DIR}
	mv *${FINAL_NAME}*.zip ${LOCAL_IMAGE_DIR}
	echo "${BUILD_URL}" > ${LOCAL_IMAGE_DIR}/BUILD_INFO.txt

	#重复APK检查
	if [ ! -e "${LOCAL_IMAGE_DIR}/repeat_apk.txt" ]; then
		cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH}
		if [ -e "cache/error.log" ]; then
			cp -rf cache/error.log out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}/${LOCAL_IMAGE_DIR}/repeat_apk.txt
			REPEAT_APK="存在重复APK，详情参阅下文"
			echo "REPEAT_APK='$REPEAT_APK'" >> ${PACKAGE_INFO_COLLECT_FILE}
		else
			echo "NA" > out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}/${LOCAL_IMAGE_DIR}/repeat_apk.txt
			REPEAT_APK="没有重复APK"
			echo "REPEAT_APK='$REPEAT_APK'" >> ${PACKAGE_INFO_COLLECT_FILE}
		fi
	fi
}

#推送正式版本到svn
function push_image_to_svn()
{
	#$1 LOCAL_IMAGE_DIR

	TMP_PATH=${PACK_WORK_DIR}/out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}/
	cd ${PACK_WORK_DIR}/out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}/
	if [ ${SELECT_PUSH_SVN} == 'true' ]; then
		push_to_svn ${TMP_PATH} ${1} ${SVN_IMAGE_UPLOAD_PATH}
		SVN_IMAGE_UPLOAD_DIR_PATH=${SVN_IMAGE_UPLOAD_PATH}/${1}
		echo "SVN_IMAGE_UPLOAD_DIR_PATH='$SVN_IMAGE_UPLOAD_DIR_PATH'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	else
		echo "SVN_IMAGE_UPLOAD_DIR_PATH='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	fi
}

#上传到指定用户目录
function upload_base_zip_server()
{
	#$1 LOCAL_IMAGE_DIR
	#$2 FINAL_NAME_USER
	#$3 FINAL_NAME_USERDEBNUG
	echo "upload_base_zip_server start"
	cd ${PACK_WORK_DIR}/out/otapackages/${PACK_PARAS_PLATFORM}/${PACK_PARAS_PROJECT}/

	if [ -e $1 ]; then
		cd ${PACK_WORK_DIR}/out/base-otapackage
		BASEPACKAGE_user=`ls |grep ${2} |grep "BASEPACKAGE" |head -n 1`
		BASEPACKAGE_userdebug=`ls |grep ${3} |grep "BASEPACKAGE" |head -n 1`
		len1=${#BASEPACKAGE_user}
		len2=${#BASEPACKAGE_userdebug}
		if [ $len1 -gt 0 ]; then
			sshpass -p ${PUSH_CUSTOM_PASSWORD} scp ${BASEPACKAGE_user} ${PUSH_CUSTOM_DIR}
		fi
		if [ $len2 -gt 0 ]; then
			sshpass -p ${PUSH_CUSTOM_PASSWORD} scp ${BASEPACKAGE_userdebug} ${PUSH_CUSTOM_DIR}
		fi
	else
		exit 1
	fi
	echo "upload_base_zip_server end"
}

function collect_package_info()
{

	#$1	$JENKINS_AUTOBUILD_SDK_PATH
	#$2 $JENKINS_AUTOBUILD_SDK
	#$3 $BUILD_TYPE
	#####基础信息收集
	BUILD_TYPE=$3
	echo "SDK_BUILD='$SDK_BUILD'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	echo "BUILD_TYPE='$BUILD_TYPE'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	echo "PATCH_LIST='$PATCH_LIST'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	#安全补丁
	SECURITY_PATCH=`cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH} && cat cache/target_files/SYSTEM/build.prop | grep 'ro.build.version.security_patch'|sed '/#/d'|awk -F '=' '{print $NF}'`
	echo "SECURITY_PATCH='$SECURITY_PATCH'" >> ${AUTOBUILD_INFO_COLLECT_FILE}

	#版本号
	BUILD_VER=`cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH} && cat cache/target_files/SYSTEM/build.prop | grep 'ro.odm.sdmc.firmware.ver'|sed '/#/d'|awk -F '=' '{print $NF}'`
	echo "BUILD_VER='$BUILD_VER'" >> ${AUTOBUILD_INFO_COLLECT_FILE}

	#强制patch版本
	MANDATORY_PATCH=`cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH} && cat cache/target_files/VENDOR/build.prop | grep 'ro.odm.sdmc.mandatory'|awk -F '=' '{print $NF}'`
	if [ -n "$MANDATORY_PATCH" ]; then
		echo "MANDATORY_PATCH='$MANDATORY_PATCH'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	else
		echo "MANDATORY_PATCH='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	fi

	#GTV版本：
	GTV_VERSION=`cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH} && cat cache/target_files/VENDOR/build.prop | grep 'ro.com.google.gmsversion'|sed '/#/d'|awk -F '=' '{print $NF}'`
	echo "GTV_VERSION='$GTV_VERSION'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	#sdmc-lib commit信息
	#sdmc_lib_commit_info $1

	#package tools commit信息
	package_tool_commit_info ${PACKAGE_TOOLS}


	#amlogic_s_stb最新提交和分支信息
	if [ "$SDK_BUILD" == "true" ]; then
		cd ${2}/sdk_patch/amlogic_s_stb
		PATCH_BRANCH=`cat .git/HEAD | awk -F '/' '{print $3}'`
		PATCH_TOOL_COMMIT_INFO=`git log --oneline | sed -n '1p'`
		echo "PATCH_BRANCH='$PATCH_BRANCH'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
		echo "PATCH_TOOL_COMMIT_INFO=$PATCH_TOOL_COMMIT_INFO" >> ${BUILD_SELF_PROPERTIES_FILE}
		cd -
	else
		echo "PATCH_BRANCH='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
		echo "PATCH_TOOL_COMMIT_INFO='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	fi 

	#打包指纹：
	FINGERPRINT=`cd ${WORKSPACE}/${PACK_WORK_RELATIVE_PATH} && cat cache/target_files/SYSTEM/build.prop | grep 'ro.system.build.fingerprint'|sed '/#/d'|awk -F '=' '{print $2}'`
	echo "FINGERPRINT='$FINGERPRINT'" >> ${PACKAGE_INFO_COLLECT_FILE}

	#svn最新提交
	#${PROJECT_URL}
	tmp1=`svn log -l 1 ${PROJECT_URL} --username ${SVN_USERNAME} --password ${SVN_PASSWORD}`
	tmp_commit_id=$(echo ${tmp1} | cut -d ' ' -f 2)
	tmp_name=$(echo ${tmp1} | cut -d '|' -f 2)
	tmp_commit=$(echo ${tmp1} | sed '1,2d'| head -n -2)
	PACKAGE_CUSTOM_COMMIT_INFO="$tmp_commit_id $tmp_name $tmp_commit"
	echo "PACKAGE_CUSTOM_COMMIT_INFO='$PACKAGE_CUSTOM_COMMIT_INFO'" >> ${PACKAGE_INFO_COLLECT_FILE}
}

#sdmc-lib仓库提交信息获取
function sdmc_lib_commit_info()
{
	init_auto_build_env
	if [ ${SDK_BUILD} == true ];then
		cd $1/vendor/sdmc/sdmc-libs
		SDMC_LIB_INFO=`git log --oneline | sed -n '1p'`
		echo "SDMC_LIB_INFO='$SDMC_LIB_INFO'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	else
		echo "SDMC_LIB_INFO='NA'" >> ${AUTOBUILD_INFO_COLLECT_FILE}
	fi
}

#package_tool更新信息获取
function package_tool_commit_info()
{
	cd $1
	PACKAGE_TOOL_COMMIT_INFO=`git log --oneline | sed -n '1p'`
	echo "PACKAGE_TOOL_COMMIT_INFO='$PACKAGE_TOOL_COMMIT_INFO'" >> ${PACKAGE_INFO_COLLECT_FILE}
}

```

# 5. 项目构建参数页面

用于实现每个项目不同的参数选择

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/39d3327c-a987-49ab-90cb-ae01d6b64741.png)

项目构建参数的实现页面写在 配置 中，这些参数都是创建项目后填写一次，后续基本无需修改

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/d3613aad-8824-46f0-8959-e49af8df4824.png)

在参数设置中，我们可以添加想要的参数，添加的参数根据不同的类型会显示在构建状态页面

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/978ad437-2a7e-4d85-9b87-5aad3e83962d.png)

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/a7276c75-65f6-4ad2-ac21-cc32981c47e0.png)

在Prepare an environment for the run 中，填写打包工程需要用到的参数

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/2760f9ff-0a11-473a-81ff-736f48bd6be6.png)

流水线部分，填写我们的主要的pipeline脚本，在我们jenkins运行时，会按照流水线中的pipeline脚本去进行执行，并调用SDMC脚本，去进行编译或者打包

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/019991dc-06ca-4693-b0b9-8e6320e3f951.png)

状态集的实现是在pipeline脚本中

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/bbbff26a-22b1-48a9-9abb-49746553a36a.png)

## 5.1 创建项目

参考文档：

[Jenkins维护说明文档](https://alidocs.dingtalk.com/i/nodes/KGZLxjv9VG37r3ajTl6m96XyV6EDybno?utm_scene=team_space)

# 6. 使用说明

首先要确保项目正确的创建

## 6.1 必填参数部分

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/83ccc90e-b456-4c7e-9f1a-975b3a0d5a0b.png)

### SDK Build

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/3e28c970-9015-4a6f-805c-3873de118f4c.png)

是否选择sdk build 也就是auto build，将会影响后续 SDK编译参数配置 和 版本构建参数配置 的选择

如果勾选，后续模块只用填写SDK编译参数配置

如果不勾选，后续模块只用填写版本构建参数配置

### BUILD_VERSION 和 USE_NEW_VISION

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/061e4912-bc48-4952-a66a-e0710817bd22.png)

这个是使用新旧版本号命名规范的区别

使用新的版本号命名规范时，BUILD_VERSION中填写的版本号将不会生效

### SELECT_PUSH_SVN

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/9e30c521-5528-41dc-a35c-a4e8dfc9469f.png)

这个选项是，是否将打包后的正式软件上传svn

## 6.2 SDK编译参数配置（勾选SDK_BUILD时生效）

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/3ebe12af-5444-40a2-b155-1b5df1d24353.png)

### CHOSE_BUILD_TYPE 和 BUILD_COVER

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/6bca95a9-c12d-46ab-9dfd-f324ceab542b.png)

这个两个选项是，在选择sdk build 时，CHOSE_BUILD_TYPE会选择sdk build 编译的类型（user，userdebug，both），BUILD_COVER会选择代码的拉取形式（cover，create）

### ROLL_BACK 和 ROOL_BACK_FILE_PATH

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/651d28ea-b085-4d4d-8b4f-2c8635037564.png)

这里是启用版本回退功能，启用的话必须要填写版本回退的xml路径，这里需要填写的是svn的路径，需要具体到是某一个xml

## 6.3 版本构建参数配置（取消勾选SDK_BUILD时生效）

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/c855d408-6d81-47c4-b106-86d7511ca4df.png)

### SELECT_PACKAGE

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/9db624cb-afdc-44f8-a56e-96fc5f329519.png)

此处是选择打包材料的类型，

如果选择user/userdebug的材料，将会使用SHOW_PROJECT_PACKAGE_PATH中展示的user/userdebug材料

如果选择custom，将会使用CUSTOM_PACKAGE_PATH中填写的材料路径

### SELECT_PUSH_CUSTOM

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLPW7jRa0Opz2/img/65de0a3f-76f3-4023-be65-5523e340f02c.png)

勾选此处会将base包拷贝到填写的本地路径上