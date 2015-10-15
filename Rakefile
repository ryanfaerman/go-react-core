require 'fileutils'

APP_NAME = 'go-react-core'

desc "Build for release"
task :release do
  exit(1) unless system %Q{GOOS=linux GOARCH=amd64 go build -o "bin/#{APP_NAME}" -ldflags "#{ldflags}"}
end

desc "Build to run locally"
task :build do
  cancel_clean('Halting build...')
  puts "Building #{APP_NAME} #{version} ..."
  exit(1) unless system %Q{go build -ldflags "#{ldflags}"}
end

desc "Clean packages created in bin"
task :clean do
  system "rm -rf ./bin/"
end

namespace :frontend do
  task :build do
    cancel_clean('Halting frontend build...')
    puts "Building #{APP_NAME} #{version} frontend ..."
    exit(1) unless system %Q{cd frontend && npm run dist}
  end
end

desc "Build and Run"
task :run => [:build, 'frontend:build'] do
  cancel_clean('Shutting down...')
  puts 'Starting application...'
  run('./' + File.basename(FileUtils.pwd))
end


def version
  ENV['VERSION'] || `git describe --tags --dirty --always`.chomp
end

def cancel_clean(msg)
   # Trap ctrl-c and display a nice message instead of Rake barfing
  # a stack trace into our terminal.
  Signal.trap('SIGINT') do
    puts "\n#{msg}"
    exit(0)
  end
end

def ldflags
  if go15?
    "-X main.Version=#{version}"
  else
    "-X main.Version '#{version}'"
  end
end

def go15?
  !!(`go version` =~ /go1.5/)
end

def run(cmd, &blk)
  subprocess(cmd) do |stdout, stderr, thread|
    if stdout
      if blk.nil?
        puts stdout
      else
        blk.call(stdout)
      end
    end
    puts stderr if stderr
  end
end

def subprocess(cmd, &block)
  # see: http://stackoverflow.com/a/1162850/83386
  Open3.popen3(cmd) do |stdin, stdout, stderr, thread|
    # read each stream from a new thread
    { :out => stdout, :err => stderr }.each do |key, stream|
      Thread.new do
        until (line = stream.gets).nil? do
          # yield the block depending on the stream
          if key == :out
            yield line, nil, thread if block_given?
          else
            yield nil, line, thread if block_given?
          end
        end
      end
    end

    thread.join # don't exit until the external process is done
  end
end
