stage("Sanity Check") {
  node('master') {
    checkout scm
    sh 'git submodule update --init'
    // sh 'tests/ci_build/ci_build.sh lint make cpplint'
    // sh 'tests/ci_build/ci_build.sh lint make rcpplint'
    // sh 'tests/ci_build/ci_build.sh lint make jnilint'
    // sh 'tests/ci_build/ci_build.sh lint make pylint'
  }
}

def mx_lib = 'lib/libmxnet.so'

def pack_lib(name, mx_lib) {
  sh """
echo "Packing ${mx_lib} into ${name}"
md5sum ${mx_lib}
"""
  stash includes: mx_lib, name: name
}

def unpack_lib(name, mx_lib) {
  unstash name
  sh """
echo "Unpacking ${mx_lib} from ${name}"
md5sum ${mx_lib}
"""
}


stage('Build') {
  parallel 'CPU': {
    node {
      ws('workspace/cpu-build') {
        checkout scm
        sh 'git submodule update --init'
        sh '''tests/ci_build/ci_build.sh cpu  make -j$(nproc) USE_BLAS=openblas'''
        pack_lib 'cpu', mx_lib
      }
    }
  },
  'GPU: CUDA7.5+cuDNN5': {
    node('GPU') {
      ws('workspace/gpu-build') {
      checkout scm
      sh 'git submodule update --init'
//      sh '''tests/ci_build/ci_build.sh gpu make -j$(nproc) \
//USE_CUDA=1 \
//USE_CUDA_PATH=/usr/local/cuda \
//USE_CUDNN=1 \
//USE_BLAS=openblas \
//EXTRA_OPERATORS=example/ssd/operator
//      '''
      // sh "sleep 2; date >${mx_lib}"
      // pack_lib 'gpu', mx_lib
      }
    }
  }
}

stage('Unit Test') {
  parallel 'CPU: Python2/3': {
    node {
      echo "python2"
      unpack_lib 'cpu', mx_lib
    }
  },
  'GPU: Python2/3': {
    node {
      echo "python3"
      unpack_lib 'cpu', mx_lib
      // unpack_lib 'gpu', mx_lib
    }
  },
  'Scala': {
    node {
      echo "xxx"
    }
  }
}
