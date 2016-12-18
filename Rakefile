task :build_tools do
  sh '(cd buildtools; ansible-container build)'
end

task :war do
  sh './mvnw war'
end
