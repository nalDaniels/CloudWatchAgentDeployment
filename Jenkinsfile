pipeline {
agent any
stages {
stage ('Build') {
steps {
sh '''#!/bin/bash
python3 -m venv test3
source test3/bin/activate
pip install pip --upgrade
pip install -r requirements.txt
export FLASK_APP=application
'''
}
}
stage ('test') {
steps {
sh '''#!/bin/bash
source test3/bin/activate
py.test --verbose --junit-xml test-reports/results.xml
'''
}
post{
always {
junit 'test-reports/results.xml'
}
}
}
stage ('Clean') {
steps {
sh '''#!/bin/bash
if [[ $(ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2) != 0 ]]
then
ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2 > pid.txt
kill $(cat pid.txt)
exit 0
fi
'''
}
}
stage ('Deploy') {
steps {
keepRunning {
sh '''#!/bin/bash
sudo pip install -r requirements.txt
sudo python3 -m pip install ddtrace
sudo export DD_SERVICE=”url-app”
sudo export DD_ENV=”Prod”
sudo export DD_VERSION=”1.0”
sudo export DD_PROFILING_ENABLE=true
sudo export DD_LOGS_INJECTION=true
sudo export DD_URL=http://18.209.157.35:8000
sudo pip install gunicorn
sudo python3 -m ddtrace-run gunicorn -w 4 application:app -b 0.0.0.0 --daemon
'''
}
}
}
}
}
