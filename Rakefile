require 'json'
require 'pry'
require 'backticks'

STDOUT.sync = true
DOCKERBIN = 'docker-compose'.freeze

DEBUG = ENV['DEBUG'] ? true : false

def stream_command(cmd)
  status = Backticks::Runner.new(interactive: true).run(cmd).join.status
  status.to_i == 0
end

def docker(cmd)
  stream_command("#{DOCKERBIN} #{cmd}")
end

namespace :cluster do
  desc 'Build custom artifacts'
  task :build_artifacts do
    stream_command("cd container_definitions && packer build 01-container-base.json")
  end

  %w{build config events up down scale logs down kill}.each do |op|
    desc "Pass through to docker-compose"
    task op.to_sym, [:args] do |_, args|
      begin
        case op
        when 'up' || 'start'
          Rake::Task['cluster:build'].invoke
          args = "-d #{args}"
        when 'build'
          Rake::Task['cluster:build_artifacts'].invoke
        end
        docker "#{op} #{args}"
      rescue SignalException => e
        puts "Got exception #{e}, undoing current operation..."
        newop = case op
        when 'up'
          'down'
        when 'start'
          'stop'
        else
          nil
        end
        docker(newop) if newop
      end
    end
  end
end

namespace :test do
  desc 'Test building a cluster'
  task :build do
    %w{build}.each do |op|
      Rake::Task["cluster:#{op}"].invoke
    end
  end
end
