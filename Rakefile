task :build_tools do
  sh '(cd ansible/buildtools; ansible-container build --from-scratch)'
end

task :build do
  sh './mvnw package'
end

task :checkstyle do
  sh './mvnw checkstyle:checkstyle'
end

task :pmd do
  sh './mvnw pmd:pmd'
end

task :docker_build do
  sh 'cp target/springboot-petclinic-1.4.2.jar docker/petclinic.jar'
  sh 'docker build -t rfhayashi/petclinic docker'
end

task :docker_push do
  sh 'docker login -u $USERNAME -p $PASSWORD'
  sh 'docker push rfhayashi/petclinic'
end

task :run_docker => :build_docker do
  sh 'docker run -d -p 8080:8080 petclinic'
end

task :deploy do
  sh 'vagrant plugin install vagrant-aws'
  sh '(cd aws; vagrant up --provider=aws)'
  sh '(cd aws; vagrant ssh -c "docker run -d -p 8080:8080 rfhayashi/petclinic")'
end

task :undeploy do
  sh '(cd aws; vagrant destroy --force)'
end
