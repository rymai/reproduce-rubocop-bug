## Reproduction of a RuboCop bug when `require` is from `inherit_from`/`inherit_gem`

- `Rails/FilePath` exists in `rubocop` and defaults to `Enabled: true`
- `RSpec/FilePath` exists in `rubocop-rspec`
- You either have a file or a gem with
  ```yml
  require:
    - rubocop-rspec
  ```
- You want to disable the `RSpec/FilePath` in your final RuboCop config file:
  ```yml
  inherit_from:
    - ./.inherited-rubocop.yml

  RSpec/FilePath:
    Enabled: false
  ```
  or
  ```yml
  inherit_gem:
    rubocop-inheritance-bug-gem:
      - rubocop.yml

  RSpec/FilePath:
    Enabled: false
  ```

The rule isn't disabled:

```
$ bundle exec rubocop -c .rubocop-inherit-from-file.yml --show-cops
.rubocop-inherit-from-file.yml: RSpec/FilePath has the wrong namespace - should be Rails
RSpec/FilePath:
  Description: Checks that spec file paths are consistent with the test subject.
  Enabled: true
  CustomTransform:
    RuboCop: rubocop
    RSpec: rspec
  IgnoreMethods: false
  StyleGuide: http://www.rubydoc.info/gems/rubocop-rspec/RuboCop/Cop/RSpec/FilePath
```

```
$ bundle exec rubocop -c .rubocop-inherit-from-gem.yml --show-cops
.rubocop-inherit-from-gem.yml: RSpec/FilePath has the wrong namespace - should be Rails
RSpec/FilePath:
  Description: Checks that spec file paths are consistent with the test subject.
  Enabled: true
  CustomTransform:
    RuboCop: rubocop
    RSpec: rspec
  IgnoreMethods: false
  StyleGuide: http://www.rubydoc.info/gems/rubocop-rspec/RuboCop/Cop/RSpec/FilePath
```

Note the warning:

```
.rubocop-inherit-from-file.yml: RSpec/FilePath has the wrong namespace - should be Rails
```

## Work-around

You can work-around this bug by re-requiring `rubocop-rspec` in the final RuboCop config file:

```yml
require:
  - rubocop-rspec

inherit_from:
  - ./.inherited-rubocop.yml

RSpec/FilePath:
  Enabled: false
```
or
```yml
require:
  - rubocop-rspec

inherit_gem:
  rubocop-inheritance-bug-gem:
    - rubocop.yml

RSpec/FilePath:
  Enabled: false
```

The rule is now disabled properly:

```
› bundle exec rubocop -c .rubocop-inherit-from-file-with-require.yml --show-cop
RSpec/FilePath:
  Description: Checks that spec file paths are consistent with the test subject.
  Enabled: false
  CustomTransform:
    RuboCop: rubocop
    RSpec: rspec
  IgnoreMethods: false
  StyleGuide: http://www.rubydoc.info/gems/rubocop-rspec/RuboCop/Cop/RSpec/FilePath
```

```
› bundle exec rubocop -c .rubocop-inherit-from-gem-with-require.yml --show-cops
RSpec/FilePath:
  Description: Checks that spec file paths are consistent with the test subject.
  Enabled: false
  CustomTransform:
    RuboCop: rubocop
    RSpec: rspec
  IgnoreMethods: false
  StyleGuide: http://www.rubydoc.info/gems/rubocop-rspec/RuboCop/Cop/RSpec/FilePath
```
