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
RUNASGROUP = ENV['INSPIRCD_GROUP']    || getgrnam('irc').gid rescue Process.gid.to_s
RUNASUSER  = ENV['INSPIRCD_USER']     || getpwnam('irc').uid rescue Process.uid.to_s

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
		execute! './configure', '--system', '--gid', RUNASGROUP, '--uid', RUNASUSER
		execute! 'make', "-j#{Rake::CpuCounter.count}", 'install'
	end
end

def fetch!
	chdir __dir__ do
		if File.directory? SOURCE_DIRECTORY
			chdir SOURCE_DIRECTORY do
				execute! 'git', 'clean', '-dfx'
				execute! 'git', 'reset', '--hard'
				execute! 'git', 'fetch', '--all', '--force'
				execute! 'git', 'checkout', REVISION
			end
		else
			execute! 'git', 'clone', '--branch', REVISION, GIT, SOURCE_DIRECTORY
		end
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
