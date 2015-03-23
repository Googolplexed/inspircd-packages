# 
# InspIRCd -- Internet Relay Chat Daemon
#
#   Copyright (C) 2015 Peter Powell <petpow@saberuk.com>
#
# This file is part of InspIRCd.  InspIRCd is free software: you can
# redistribute it and/or modify it under the terms of the GNU General Public
# License as published by the Free Software Foundation, version 2.
#
# This program is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
# FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more
# details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#


GIT        = ENV['INSPIRCD_GIT']      || 'git://github.com/inspircd/inspircd.git'
REVISION   = ENV['INSPIRCD_REVISION'] || 'master'
RUNASGROUP = ENV['INSPIRCD_GROUP']    || (Etc.getgrnam('irc').gid rescue Process.gid).to_s
RUNASUSER  = ENV['INSPIRCD_USER']     || (Etc.getpwnam('irc').uid rescue Process.uid).to_s

DEBIAN_DIRECTORY  = "#{__dir__}/debian"
EXTRAS_DIRECTORY  = "#{__dir__}/extras"
INSTALL_DIRECTORY = "#{__dir__}/install"
SOURCE_DIRECTORY  = "#{__dir__}/source"

def execute! *command
	puts "$ #{command.join ' '}"
	options = { 'PWD' => Dir.pwd }
	system options, *command or fail "Command '#{command.join ' '}' failed with status #{$?.exitstatus}."
end

def compile!
	rm_rf INSTALL_DIRECTORY if File.exist? INSTALL_DIRECTORY

	chdir SOURCE_DIRECTORY do
		ENV['DESTDIR'] = INSTALL_DIRECTORY
		execute! './configure', '--development', '--system', '--gid', RUNASGROUP, '--uid', RUNASUSER
		execute! 'make', "-j#{Rake::CpuCounter.count}", 'install'
	end
end

def fetch!
	chdir __dir__ do
		rm_rf SOURCE_DIRECTORY
		execute! 'git', 'clone', '--branch', REVISION, '--depth', '1', GIT, SOURCE_DIRECTORY
	end
end

def template! source, target, variables
	template = File.read source
	variables.each do |key, value|
		template.gsub! /@#{key}@/i, value
	end
	File.open target, 'w+' do |fh|
		fh.write template
	end
end

def get_version
	unless defined? $VERSION
		output = `#{SOURCE_DIRECTORY}/src/version.sh 2>/dev/null`.chomp
		fail "Unable to read version.sh!" unless $?.exitstatus == 0
		$VERSION = output.gsub /^\D+/, nil.to_s
	end
	return $VERSION
end

desc 'Purge all generated files'
task :clean do
	rm_rf SOURCE_DIRECTORY
	execute! 'git', 'clean', '-dfx'
end

desc 'Generate a Debian DEB package'
task :deb do
	require 'digest'

	fetch!
	compile!

	mkdir_p "#{INSTALL_DIRECTORY}/etc/init.d"
	mv "#{INSTALL_DIRECTORY}/var/lib/inspircd/inspircd", "#{INSTALL_DIRECTORY}/etc/init.d"

	mkdir_p "#{INSTALL_DIRECTORY}/lib/systemd/system"
	if get_version =~ /^2\.0\./
		cp "#{EXTRAS_DIRECTORY}/inspircd.service", "#{INSTALL_DIRECTORY}/lib/systemd/system"
	else
		mv "#{INSTALL_DIRECTORY}/var/lib/inspircd/inspircd.service", "#{INSTALL_DIRECTORY}/lib/systemd/system"
	end

	chdir INSTALL_DIRECTORY do
		execute! 'tar', 'cfz', "#{DEBIAN_DIRECTORY}/data.tar.gz", '.'

		File.open "#{DEBIAN_DIRECTORY}/md5sums", 'w' do |fh|
			Dir['**/**'].select { |file| File.file? file }.each do |file|
				fh.puts "#{file} #{Digest::MD5.hexdigest File.read file}"
			end
		end
	end

	chdir DEBIAN_DIRECTORY do
		architecture = `uname -p 2>/dev/null`.chomp
		architecture.gsub! /^x86_64$/, 'amd64'

		variables = { architecture: architecture, version: get_version }
		template! "#{DEBIAN_DIRECTORY}/control.in", "#{DEBIAN_DIRECTORY}/control", variables

		execute! 'tar', 'cfz', "#{DEBIAN_DIRECTORY}/control.tar.gz", 'control', 'md5sums'
		execute! 'ar', 'rc', "#{__dir__}/inspircd_#{get_version}_#{architecture}.deb", 'debian-binary', 'control.tar.gz', 'data.tar.gz'
	end
end

task :default do
	puts 'Targets:'
	`#{$PROGRAM_NAME} --tasks`.each_line do |line|
		puts " #{line}"
	end
end

desc 'Print settings collected from the environment.'
task :environment do
	puts "GIT        = #{GIT}"
	puts "REVISION   = #{REVISION}"
	puts "RUNASGROUP = #{RUNASGROUP}"
	puts "RUNASUSER  = #{RUNASUSER}"
end

desc 'Generate a Tarball package'
task :tarball do
	fetch!
	compile!

	chdir INSTALL_DIRECTORY do
		execute! 'tar', 'cfz', "#{__dir__}/InspIRCd-#{get_version}.tar.gz", '.'
	end
end
