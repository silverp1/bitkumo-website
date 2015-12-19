require 'html/proofer'

task :test do
    sh "bundle exec jekyll build"
    HTML::Proofer.new("./_site", {
        :href_ignore => [
            "#"
        ],
        :domain_ignore => [
            "linkedin.com"
        ]
        }).run
end
