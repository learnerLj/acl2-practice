# Configure the environment

```bash
#!/bin/bash
set -eux
SBCL_NAME=sbcl.tar.bz2
ACL2_PATH=$(pwd)/acl2
export PATH=$ACL2_PATH/books/build:$PATH
# lisp compile
apt-get install curl bzip2 git -y
if [! -a $SBCL_NAME]; then
    curl -o $SBCL_NAME -L https://nchc.dl.sourceforge.net/project/sbcl/sbcl/2.4.4/sbcl-2.4.4-x86-64-linux-binary.tar.bz2
fi
bzip2 -cd $SBCL_NAME | tar xvf -
cd sbcl-2.4.4-x86-64-linux && sh install.sh
# rm -rf sbcl-2.4.4-x86-64-linux $SBCL_NAME

# acl2
cd ..
if [! -d "acl2"]; then
    git clone https://github.com/acl2/acl2.git
fi
cd acl2
make LISP=sbcl
apt-get install acl2 -y

# paper code
cd books/workshops/2023/kumar-etal/
./make.sh
```# acl2-practice
