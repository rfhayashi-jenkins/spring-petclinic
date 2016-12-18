task :build_tools do
  sh '(cd buildtools; ansible-container build --from-scratch)'
end

task :build do
  sh './mvnw package'
end
