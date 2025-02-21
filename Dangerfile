# Sometimes its a README fix, or something like that - which isn't relevant for
# including in a CHANGELOG for example
has_app_changes = !git.modified_files.grep(/lib/).empty?
has_test_changes = !git.modified_files.grep(/spec/).empty?

if has_app_changes && !has_test_changes
  warn "Tests were not updated"
end

# Make a note about contributors not in the organization
unless github.api.organization_member?('danger', github.pr_author)
  message "@#{github.pr_author} is not a contributor yet, would you like to join the Danger org?"

  # Pay extra attention if they modify the gemspec
  if git.modified_files.include?("*.gemspec")
    warn "External contributor has edited the Gemspec"
  end
end

# Mainly to encourage writing up some reasoning about the PR, rather than
# just leaving a title
if github.pr_body.length < 5
  fail "Please provide a summary in the Pull Request description"
end

# Let people say that this isn't worth a CHANGELOG entry in the PR if they choose
declared_trivial = (github.pr_title + github.pr_body).include?("#trivial") || !has_app_changes

if !git.modified_files.include?("CHANGELOG.md") && !declared_trivial
  fail("Please include a CHANGELOG entry. \nYou can find it at [CHANGELOG.md](https://github.com/danger/danger/blob/master/CHANGELOG.md).")
end

# Docs are critical, so let's re-run the docs part of the specs and show any issues:
core_plugins_docs = `bundle exec danger plugins lint lib/danger/danger_core/plugins/*.rb --warnings-as-errors`

# If it failed, fail the build, and include markdown with the output error.
unless $?.success?
  # We want to strip ANSI colors for our markdown, and make paths relative
  colourless_error = core_plugins_docs.gsub(/\e\[(\d+)(;\d+)*m/, "")
  markdown("### Core Docs Errors \n\n#{colourless_error}")
  fail("Failing due to documentation issues, see below.", sticky: false)
end

# Oddly enough, it's quite possible to do some testing of Danger, inside Danger
# So, you can ignore these, if you're looking at the Dangerfile to get ideas.
#
# If these are all empty something has gone wrong, better to raise it in a comment
if git.modified_files.empty? && git.added_files.empty? && git.deleted_files.empty?
  fail "This PR has no changes at all, this is likely an issue during development."
end

# This comes from `./danger_plugins/protect_files.rb` which is automatically parsed by Danger
files.protect_files(path: "danger.gemspec", message: ".gemspec modified", fail_build: false)

# Ensure that our core plugins all have 100% documentation
core_plugins = Dir.glob("lib/danger/danger_core/plugins/*.rb")
core_lint_output = `bundle exec yard stats #{core_plugins.join ' '} --list-undoc --tag tags`

if !core_lint_output.include?("100.00%")
  fail "The core plugins are not at 100% doc'd - see below:", sticky: false
  markdown "```\n#{core_lint_output}```"
elsif core_lint_output.include? "warning"
  warn "The core plugins are have yard warnings - see below", sticky: false
  markdown "```\n#{core_lint_output}```"
end

junit.parse "junit-results.xml"
junit.headers = [:file, :name]
junit.report