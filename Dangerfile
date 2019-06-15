message "Build Steps succeded ✅"

# Metadata Validations
## PR Title Validation
warn "Please provide a valid Pull Request title (ex: \"IOX-01: Title\")" if (gitlab.mr_title =~ /^[[:alpha:]]{2,}\-\d{1,4}/).nil?

## Commit validation title
if !git.commits.any? { |c| c.message =~ /^[[:alpha:]]{2,}\-\d{1,4}/ || c.message =~ /^Merge branch/ }
  warn "Please ensure your commits match the required valid pattern (ex: \"IOX-01 Title\")"
end

## PR Description Validation
warn "Please provide a summary in the Pull Request description" if gitlab.mr_body.size < 5

## Labels Validation
if gitlab.mr_labels.empty?
  warn "Please add labels to this merge request."
end

# Content Validations
## Podspec Validation
if git.modified_files.grep(/\.podspec/).empty?
  warn('Don\'t forget to update the Podspec 👍')
end

## Podfile Validation
if git.modified_files.include?("Podfile") && !git.modified_files.include?("Podfile.lock")
  warn('You must commit the Podfile.lock when updating the Podfile')
end

## Gemfile Validation
if git.modified_files.include?("Gemfile") && !git.modified_files.include?("Gemfile.lock")
  warn('You must commit the Gemfile.lock when updating the Gemfile')
end

## Info.plist file shouldn't change often
if !git.modified_files.grep(/Info.plist/).empty?
  warn "Info.plist file shouldn't change often"
end

## SwiftLint Validation
swiftlint.lint_files

## Checking if spec dependencies were updated (only on slices)
if git.modified_files.include?("Podfile") && git.modified_files.grep(/\Slice.podspec/).empty?

  Dir.entries(".").reject{|f| File.directory? f}.each do |curr_dir_file|
    next unless curr_dir_file.match(/[a-zA-Z]+Slice\.podspec$/)

    dependencies = {}

    File.read(curr_dir_file).lines do |line|
      next unless match = line.match(/spec.dependency\s?'(.*)',\s?'(.*)'/)

      spec, version = match.captures
      dependencies[spec] = version
    end

    File.read("Podfile").lines do |line|
      next unless match = line.match(/pod\s?'(.*)',\s?'(.*)'/)

      pod, version = match.captures

      if dependencies.key?(pod) && dependencies[pod] != version
        warn "The #{pod} dependency was not updated. Podspec has '#{dependencies[pod]}' and the podfile has '#{version}'"
      end

    end

    break
  end
end
