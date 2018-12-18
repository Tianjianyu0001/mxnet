def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, yaml: """
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: jenkins-slave 
spec:
  containers:
  - name: mxnet
    image: docker.hobot.cc/dlp/mxnet:runtime-py3.6-cudnn7.3-cuda9.2-centos7
"""
) {
    node (label) {
        stage("clone mxnet"){
            container("mxnet"){
                sh """
                cd /home/jenkins/workspace
                git clone --recursive https://github.com/Tianjianyu0001/incubator-mxnet.git
                """
            }
        }
        stage("mxnet configure"){
            container("mxnet"){
                sh """
                cd /home/jenkins/workspace/incubator-mxnet
                sed -i "s/USE_CUDA = 0/USE_CUDA = 1/g" make/config.mk
                sed -i "s#USE_CUDA_PATH = NONE#USE_CUDA_PATH = /usr/local/cuda-9.2#g" make/config.mk
                sed -i "s/USE_CUDNN = 0/USE_CUDNN = 1/g" make/config.mk
                sed -i "s/USE_BLAS = atlas/USE_BLAS = openblas/g" make/config.mk
                sed -i "s#ADD_CFLAGS =#ADD_CFLAGS += -I/usr/include/openblas/#g" make/config.mk
                sed -i "s/USE_HDFS = 0/USE_HDFS = 1/g" make/config.mk
                """
            }
        }
        stage("mxnet path"){
            container("mxnet"){
                sh """
                echo "export JAVA_HOME=/usr/lib/jvm/java-1.7.0" >> /root/.bashrc
                echo "export HADOOP_PREFIX=/App/hadoop-2.7.2" >> /root/.bashrc
                echo "export HADOOP_HOME=\$HADOOP_PREFIX" >>/root/.bashrc
                echo "export CLASSPATH=\$HADOOP_PREFIX/lib/classpath_hdfs.jar" >> /root/.bashrc
                echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:\$HADOOP_PREFIX/lib/native:\$JAVA_HOME/jre/lib/amd64/server" >> /root/.bashrc
                echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/cuda-9.2/lib64" >> /root/.bashrc
                echo "export PATH=\$PATH:/usr/local/cuda-9.2/bin" >> /root/.bashrc
                """
            }
        }
        stage("mxnet make"){
            container("mxnet"){
                sh """
                cd /home/jenkins/workspace/incubator-mxnet
				source /root/.bashrc
                make -j8
                """
            }
        }
        stage("mxnet inspect"){
            container("mxnet"){
                sh """
                cd /home/jenkins/workspace/incubator-mxnet
                find . -name libmxnet.so
                """
            }
        }
    }
}