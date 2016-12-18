task :build_tools do
  sh '(cd ansible/buildtools; ansible-container build --from-scratch)'
end

task :build do
  sh './mvnw package'
end

task :build_docker do
  sh '(cd ansible/petclinic; ansible-container build --from-scratch)'
end

task :run_docker => :build_docker do
  sh '(cd ansible/petclinic; ansible-container run)'
end
