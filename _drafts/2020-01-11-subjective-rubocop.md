So, I had an (ambitious, one might say... or crazy) idea of doing a subjective review of ALL currently existing cops and trying to restructure them in a way that corresponds to their significance, as I feel it. As I've already told, the post is highly subjective.

Funny thing there are a lot of cops which have a settings for all imaginable styles (and some unimaginable), meaning somebody _requested that style, because they use it_. This obviously undermines value of "comon style guide" in a mature language community.

Preparation:
```
$ whatthegem rubocop
rubocop
┃  Automatic Ruby code style checking tool. Aims to enforce the community-driven Ruby Style Guide.
┃  (https://github.com/rubocop-hq/rubocop/, https://www.rubocop.org/)

  Info

          Latest version: 0.79.0 (4 days ago)
      Installed versions: 0.57.2, 0.67.2, 0.72.0, 0.75.0, 0.77.0, 0.79.0
```
OK, I have the latest version (0.79.0) already.
```
rubocop --show-cops > cops.yml
```


## Personal style / Consistency


## Subjective

## Education

## Modern

## Errors

Bundler/DuplicatedGem: Checks for duplicate gem entries in Gemfile.
Gemspec/DuplicatedAssignment: An attribute assignment method calls should be listed only once in a gemspec.

## Highly Possible Typos

Layout/ConditionPosition: Checks for condition placed in a confusing position relative to the keyword.


## Security

Bundler/InsecureProtocolSource: The source `:gemcutter`, `:rubygems` and `:rubyforge` are deprecated because HTTP requests are insecure. Please change your source to 'https://rubygems.org' if possible, or 'http://rubygems.org' if not.


## Bureucracy

Bundler/GemComment: Add a comment describing each gem.

## Pedantism

Bundler/OrderedGems: Gems within groups in the Gemfile should be alphabetically sorted.
Gemspec/OrderedDependencies: Dependencies in the gemspec should be alphabetically sorted.
Layout/ClassStructure: Enforces a configured order of definitions within a class body.


## Readability

Layout/AccessModifierIndentation: Check indentation of private/protected visibility modifiers.
Layout/ArgumentAlignment: Align the arguments of a method call if they span more than one line.
Layout/ArrayAlignment: Align the elements of an array literal if they span more than one line.
Layout/AssignmentIndentation: Checks the indentation of the first line of the right-hand-side of a multi-line assignment.
Layout/BlockAlignment: Align block ends correctly.
Layout/BlockEndNewline: Put end statement of multiline block on its own line.
Layout/CaseIndentation: Indentation of when in a case/when/[else/]end.
Layout/ClosingHeredocIndentation: Checks the indentation of here document closings.
Layout/ClosingParenthesisIndentation: Checks the indentation of hanging closing parentheses.
Layout/CommentIndentation: Indentation of comments consistent with next lines.
Layout/DefEndAlignment: Align ends corresponding to defs correctly.

---------------------------------

Gemspec/RequiredRubyVersion: Checks that `required_ruby_version` of gemspec and `TargetRubyVersion` of .rubocop.yml are equal.
Gemspec/RubyVersionGlobalsUsage: Checks usage of RUBY_VERSION in gemspec.
Layout/DotPosition: Checks the position of the dot in multi-line method calls.
Layout/ElseAlignment: Align elses and elsifs correctly.
Layout/EmptyComment: Checks empty comment.
Layout/EmptyLineAfterGuardClause: Add empty line after guard clause.
Layout/EmptyLineAfterMagicComment: Add an empty line after magic comments to separate them from code.
Layout/EmptyLineBetweenDefs: Use empty lines between defs.
Layout/EmptyLines: Don't use several empty lines in a row.
Layout/EmptyLinesAroundAccessModifier: Keep blank lines around access modifiers.
Layout/EmptyLinesAroundArguments: Keeps track of empty lines around method arguments.
Layout/EmptyLinesAroundBeginBody: Keeps track of empty lines around begin-end bodies.
Layout/EmptyLinesAroundBlockBody: Keeps track of empty lines around block bodies.
Layout/EmptyLinesAroundClassBody: Keeps track of empty lines around class bodies.
Layout/EmptyLinesAroundExceptionHandlingKeywords: Keeps track of empty lines around exception handling keywords.
Layout/EmptyLinesAroundMethodBody: Keeps track of empty lines around method bodies.
Layout/EmptyLinesAroundModuleBody: Keeps track of empty lines around module bodies.
Layout/EndAlignment: Align ends correctly.
Layout/EndOfLine: Use Unix-style line endings.
Layout/ExtraSpacing: Do not use unnecessary spacing.
Layout/FirstArgumentIndentation: Checks the indentation of the first argument in a method call.
Layout/FirstArrayElementIndentation: Checks the indentation of the first element in an array literal.
Layout/FirstArrayElementLineBreak: Checks for a line break before the first element in a multi-line array.
Layout/FirstHashElementIndentation: Checks the indentation of the first key in a hash literal.
Layout/FirstHashElementLineBreak: Checks for a line break before the first element in a multi-line hash.
Layout/FirstMethodArgumentLineBreak: Checks for a line break before the first argument in a multi-line method call.
Layout/FirstMethodParameterLineBreak: Checks for a line break before the first parameter in a multi-line method parameter definition.
Layout/FirstParameterIndentation: Checks the indentation of the first parameter in a method definition.
Layout/HashAlignment: Align the elements of a hash literal if they span more than one line.
Layout/HeredocArgumentClosingParenthesis: Checks for the placement of the closing parenthesis in a method call that passes a HEREDOC string as an argument.
Layout/HeredocIndentation: This cop checks the indentation of the here document bodies.
Layout/IndentationConsistency: Keep indentation straight.
Layout/IndentationWidth: Use 2 spaces for indentation.
Layout/InitialIndentation: Checks the indentation of the first non-blank non-comment line in a file.
Layout/LeadingCommentSpace: Comments should start with a space.
Layout/LeadingEmptyLines: Check for unnecessary blank lines at the beginning of a file.
Layout/LineLength: Limit lines to 80 characters.
Layout/MultilineArrayBraceLayout: Checks that the closing brace in an array literal is either on the same line as the last array element, or a new line.
Layout/MultilineArrayLineBreaks: Checks that each item in a multi-line array literal starts on a separate line.
Layout/MultilineAssignmentLayout: Check for a newline after the assignment operator in multi-line assignments.
Layout/MultilineBlockLayout: Ensures newlines after multiline block do statements.
Layout/MultilineHashBraceLayout: Checks that the closing brace in a hash literal is either on the same line as the last hash element, or a new line.
Layout/MultilineHashKeyLineBreaks: Checks that each item in a multi-line hash literal starts on a separate line.
Layout/MultilineMethodArgumentLineBreaks: Checks that each argument in a multi-line method call starts on a separate line.
Layout/MultilineMethodCallBraceLayout: Checks that the closing brace in a method call is either on the same line as the last method argument, or a new line.
Layout/MultilineMethodCallIndentation: Checks indentation of method calls with the dot operator that span more than one line.
Layout/MultilineMethodDefinitionBraceLayout: Checks that the closing brace in a method definition is either on the same line as the last method parameter, or a new line.
Layout/MultilineOperationIndentation: Checks indentation of binary operations that span more than one line.
Layout/ParameterAlignment: Align the parameters of a method definition if they span more than one line.
Layout/RescueEnsureAlignment: Align rescues and ensures correctly.
Layout/SpaceAfterColon: Use spaces after colons.
Layout/SpaceAfterComma: Use spaces after commas.
Layout/SpaceAfterMethodName: Do not put a space between a method name and the opening parenthesis in a method definition.
Layout/SpaceAfterNot: Tracks redundant space after the ! operator.
Layout/SpaceAfterSemicolon: Use spaces after semicolons.
Layout/SpaceAroundBlockParameters: Checks the spacing inside and after block parameters pipes.
Layout/SpaceAroundEqualsInParameterDefault: Checks that the equals signs in parameter default assignments have or don't have surrounding space depending on configuration.
Layout/SpaceAroundKeyword: Use a space around keywords if appropriate.
Layout/SpaceAroundOperators: Use a single space around operators.
Layout/SpaceBeforeBlockBraces: Checks that the left block brace has or doesn't have space before it.
Layout/SpaceBeforeComma: No spaces before commas.
Layout/SpaceBeforeComment: Checks for missing space between code and a comment on the same line.
Layout/SpaceBeforeFirstArg: Checks that exactly one space is used between a method name and the first argument for method calls without parentheses.
Layout/SpaceBeforeSemicolon: No spaces before semicolons.
Layout/SpaceInLambdaLiteral: Checks for spaces in lambda literals.
Layout/SpaceInsideArrayLiteralBrackets: Checks the spacing inside array literal brackets.
Layout/SpaceInsideArrayPercentLiteral: No unnecessary additional spaces between elements in %i/%w literals.
Layout/SpaceInsideBlockBraces: Checks that block braces have or don't have surrounding space. For blocks taking parameters, checks that the left brace has or doesn't have trailing space.
Layout/SpaceInsideHashLiteralBraces: Use spaces inside hash literal braces - or don't.
Layout/SpaceInsideParens: No spaces after ( or before ).
Layout/SpaceInsidePercentLiteralDelimiters: No unnecessary spaces inside delimiters of %i/%w/%x literals.
Layout/SpaceInsideRangeLiteral: No spaces inside range literals.
Layout/SpaceInsideReferenceBrackets: Checks the spacing inside referential brackets.
Layout/SpaceInsideStringInterpolation: Checks for padding/surrounding spaces inside string interpolation.
Layout/Tab: No hard tabs.
Layout/TrailingEmptyLines: Checks trailing blank lines and final newline.
Layout/TrailingWhitespace: Avoid trailing whitespace.
Lint/AmbiguousBlockAssociation: Checks for ambiguous block association with method when param passed without parentheses.
Lint/AmbiguousOperator: Checks for ambiguous operators in the first argument of a method invocation without parentheses.
Lint/AmbiguousRegexpLiteral: Checks for ambiguous regexp literals in the first argument of a method invocation without parentheses.
Lint/AssignmentInCondition: Don't use assignment in conditions.
Lint/BigDecimalNew: `BigDecimal.new()` is deprecated. Use `BigDecimal()` instead.
Lint/BooleanSymbol: Check for `:true` and `:false` symbols.
Lint/CircularArgumentReference: Default values in optional keyword arguments and optional ordinal arguments should not refer back to the name of the argument.
Lint/Debugger: Check for debugger calls.
Lint/DeprecatedClassMethods: Check for deprecated class method calls.
Lint/DisjunctiveAssignmentInConstructor: In constructor, plain assignment is preferred over disjunctive.
Lint/DuplicateCaseCondition: Do not repeat values in case conditionals.
Lint/DuplicateHashKey: Check for duplicate keys in hash literals.
Lint/DuplicateMethods: Check for duplicate method definitions.
Lint/EachWithObjectArgument: Check for immutable argument given to each_with_object.
Lint/ElseLayout: Check for odd code arrangement in an else block.
Lint/EmptyEnsure: Checks for empty ensure block.
Lint/EmptyExpression: Checks for empty expressions.
Lint/EmptyInterpolation: Checks for empty string interpolation.
Lint/EmptyWhen: Checks for `when` branches with empty bodies.
Lint/EndInMethod: END blocks should not be placed inside method definitions.
Lint/EnsureReturn: Do not use return in an ensure block.
Lint/ErbNewArguments: Use `:trim_mode` and `:eoutvar` keyword arguments to `ERB.new`.
Lint/FlipFlop: Checks for flip-flops.
Lint/FloatOutOfRange: Catches floating-point literals too large or small for Ruby to represent.
Lint/FormatParameterMismatch: The number of parameters to format/sprint must match the fields.
Lint/HeredocMethodCallPosition: Checks for the ordering of a method call where the receiver of the call is a HEREDOC.
Lint/ImplicitStringConcatenation: Checks for adjacent string literals on the same line, which could better be represented as a single string literal.
Lint/IneffectiveAccessModifier: Checks for attempts to use `private` or `protected` to set the visibility of a class method, which does not work.
Lint/InheritException: Avoid inheriting from the `Exception` class.
Lint/InterpolationCheck: Raise warning for interpolation in single q strs.
Lint/LiteralAsCondition: Checks of literals used in conditions.
Lint/LiteralInInterpolation: Checks for literals used in interpolation.
Lint/Loop: Use Kernel#loop with break rather than begin/end/until or begin/end/while for post-loop tests.
Lint/MissingCopEnableDirective: Checks for a `# rubocop:enable` after `# rubocop:disable`.
Lint/MultipleComparison: Use `&&` operator to compare multiple values.
Lint/NestedMethodDefinition: Do not use nested method definitions.
Lint/NestedPercentLiteral: Checks for nested percent literals.
Lint/NextWithoutAccumulator: Do not omit the accumulator when calling `next` in a `reduce`/`inject` block.
Lint/NonDeterministicRequireOrder: Always sort arrays returned by Dir.glob when requiring files.
Lint/NonLocalExitFromIterator: Do not use return in iterator to cause non-local exit.
Lint/NumberConversion: Checks unsafe usage of number conversion methods.
Lint/OrderedMagicComments: Checks the proper ordering of magic comments and whether a magic comment is not placed before a shebang.
Lint/ParenthesesAsGroupedExpression: Checks for method calls with a space before the opening parenthesis.
Lint/PercentStringArray: Checks for unwanted commas and quotes in %w/%W literals.
Lint/PercentSymbolArray: Checks for unwanted commas and colons in %i/%I literals.
Lint/RandOne: Checks for `rand(1)` calls. Such calls always return `0` and most likely a mistake.
Lint/RedundantCopDisableDirective: Checks for rubocop:disable comments that can be removed. Note: this cop is not disabled when disabling all cops. It must be explicitly disabled.
Lint/RedundantCopEnableDirective: Checks for rubocop:enable comments that can be removed.
Lint/RedundantRequireStatement: Checks for unnecessary `require` statement.
Lint/RedundantSplatExpansion: Checks for splat unnecessarily being called on literals.
Lint/RedundantStringCoercion: Checks for Object#to_s usage in string interpolation.
Lint/RedundantWithIndex: Checks for redundant `with_index`.
Lint/RedundantWithObject: Checks for redundant `with_object`.
Lint/RegexpAsCondition: Do not use regexp literal as a condition. The regexp literal matches `$_` implicitly.
Lint/RequireParentheses: Use parentheses in the method call to avoid confusion about precedence.
Lint/RescueException: Avoid rescuing the Exception class.
Lint/RescueType: Avoid rescuing from non constants that could result in a `TypeError`.
Lint/ReturnInVoidContext: Checks for return in void context.
Lint/SafeNavigationChain: Do not chain ordinary method call after safe navigation operator.
Lint/SafeNavigationConsistency: Check to make sure that if safe navigation is used for a method call in an `&&` or `||` condition that safe navigation is used for all method calls on that same object.
Lint/SafeNavigationWithEmpty: Avoid `foo&.empty?` in conditionals.
Lint/ScriptPermission: Grant script file execute permission.
Lint/SendWithMixinArgument: Checks for `send` method when using mixin.
Lint/ShadowedArgument: Avoid reassigning arguments before they were used.
Lint/ShadowedException: Avoid rescuing a higher level exception before a lower level exception.
Lint/ShadowingOuterLocalVariable: Do not use the same name as outer local variable for block arguments or block local variables.
Lint/SuppressedException: Don't suppress exceptions.
Lint/Syntax: Checks syntax error.
Lint/ToJSON: Ensure #to_json includes an optional argument.
Lint/UnderscorePrefixedVariableName: Do not use prefix `_` for a variable that is used.
Lint/UnifiedInteger: Use Integer instead of Fixnum or Bignum.
Lint/UnreachableCode: Unreachable code.
Lint/UnusedBlockArgument: Checks for unused block arguments.
Lint/UnusedMethodArgument: Checks for unused method arguments.
Lint/UriEscapeUnescape: `URI.escape` method is obsolete and should not be used. Instead, use `CGI.escape`, `URI.encode_www_form` or `URI.encode_www_form_component` depending on your specific use case. Also `URI.unescape` method is obsolete and should not be used. Instead, use `CGI.unescape`, `URI.decode_www_form` or `URI.decode_www_form_component` depending on your specific use case.
Lint/UriRegexp: Use `URI::DEFAULT_PARSER.make_regexp` instead of `URI.regexp`.
Lint/UselessAccessModifier: Checks for useless access modifiers.
Lint/UselessAssignment: Checks for useless assignment to a local variable.
Lint/UselessComparison: Checks for comparison of something with itself.
Lint/UselessElseWithoutRescue: Checks for useless `else` in `begin..end` without `rescue`.
Lint/UselessSetterCall: Checks for useless setter call to a local variable.
Lint/Void: Possible use of operator/literal/variable in void context.
Metrics/AbcSize: A calculated magnitude based on number of assignments, branches, and conditions.
Metrics/BlockLength: Avoid long blocks with many lines.
Metrics/BlockNesting: Avoid excessive block nesting.
Metrics/ClassLength: Avoid classes longer than 100 lines of code.
Metrics/CyclomaticComplexity: A complexity metric that is strongly correlated to the number of test cases needed to validate a method.
Metrics/MethodLength: Avoid methods longer than 10 lines of code.
Metrics/ModuleLength: Avoid modules longer than 100 lines of code.
Metrics/ParameterLists: Avoid parameter lists longer than three or four parameters.
Metrics/PerceivedComplexity: A complexity metric geared towards measuring complexity for a human reader.
Migration/DepartmentName: Check that cop names in rubocop:disable (etc) comments are given with department name.
Naming/AccessorMethodName: Check the naming of accessor methods for get_/set_.
Naming/AsciiIdentifiers: Use only ascii symbols in identifiers.
Naming/BinaryOperatorParameterName: When defining binary operators, name the argument other.
Naming/BlockParameterName: Checks for block parameter names that contain capital letters, end in numbers, or do not meet a minimal length.
Naming/ClassAndModuleCamelCase: Use CamelCase for classes and modules.
Naming/ConstantName: Constants should use SCREAMING_SNAKE_CASE.
Naming/FileName: Use snake_case for source file names.
Naming/HeredocDelimiterCase: Use configured case for heredoc delimiters.
Naming/HeredocDelimiterNaming: Use descriptive heredoc delimiters.
Naming/MemoizedInstanceVariableName: Memoized method name should match memo instance variable name.
Naming/MethodName: Use the configured style when naming methods.
Naming/MethodParameterName: Checks for method parameter names that contain capital letters, end in numbers, or do not meet a minimal length.
Naming/PredicateName: Check the names of predicate methods.
Naming/RescuedExceptionsVariableName: Use consistent rescued exceptions variables naming.
Naming/VariableName: Use the configured style when naming variables.
Naming/VariableNumber: Use the configured style when numbering variables.
Security/Eval: The use of eval represents a serious security risk.
Security/JSONLoad: Prefer usage of `JSON.parse` over `JSON.load` due to potential security issues. See reference for more information.
Security/MarshalLoad: Avoid using of `Marshal.load` or `Marshal.restore` due to potential security issues. See reference for more information.
Security/Open: The use of Kernel#open represents a serious security risk.
Security/YAMLLoad: Prefer usage of `YAML.safe_load` over `YAML.load` due to potential security issues. See reference for more information.
Style/AccessModifierDeclarations: Checks style of how access modifiers are used.
Style/Alias: Use alias instead of alias_method.
Style/AndOr: Use &&/|| instead of and/or.
Style/ArrayJoin: Use Array#join instead of Array#*.
Style/AsciiComments: Use only ascii symbols in comments.
Style/Attr: Checks for uses of Module#attr.
Style/AutoResourceCleanup: Suggests the usage of an auto resource cleanup version of a method (if available).
Style/BarePercentLiterals: Checks if usage of %() or %Q() matches configuration.
Style/BeginBlock: Avoid the use of BEGIN blocks.
Style/BlockComments: Do not use block comments.
Style/BlockDelimiters: Avoid using {...} for multi-line blocks (multiline chaining is always ugly). Prefer {...} over do...end for single-line blocks.
Style/BracesAroundHashParameters: Enforce braces style around hash parameters.
Style/CaseEquality: Avoid explicit use of the case equality operator(===).
Style/CharacterLiteral: Checks for uses of character literals.
Style/ClassAndModuleChildren: Checks style of children classes and modules.
Style/ClassCheck: Enforces consistent use of `Object#is_a?` or `Object#kind_of?`.
Style/ClassMethods: Use self when defining module/class methods.
Style/ClassVars: Avoid the use of class variables.
Style/CollectionMethods: Preferred collection methods.
Style/ColonMethodCall: Do not use :: for method call.
Style/ColonMethodDefinition: Do not use :: for defining class methods.
Style/CommandLiteral: Use `` or %x around command literals.
Style/CommentAnnotation: Checks formatting of special comments (TODO, FIXME, OPTIMIZE, HACK, REVIEW).
Style/CommentedKeyword: Do not place comments on the same line as certain keywords.
Style/ConditionalAssignment: Use the return value of `if` and `case` statements for assignment to a variable and variable comparison instead of assigning that variable inside of each branch.
Style/ConstantVisibility: Check that class- and module constants have visibility declarations.
Style/Copyright: Include a copyright notice in each file before any code.
Style/DateTime: Use Time over DateTime.
Style/DefWithParentheses: Use def with parentheses when there are arguments.
Style/Dir: Use the `__dir__` method to retrieve the canonicalized absolute path to the current file.
Style/Documentation: Document classes and non-namespace modules.
Style/DocumentationMethod: Checks for missing documentation comment for public methods.
Style/DoubleCopDisableDirective: Checks for double rubocop:disable comments on a single line.
Style/DoubleNegation: Checks for uses of double negation (!!).
Style/EachForSimpleLoop: Use `Integer#times` for a simple loop which iterates a fixed number of times.
Style/EachWithObject: Prefer `each_with_object` over `inject` or `reduce`.
Style/EmptyBlockParameter: Omit pipes for empty block parameters.
Style/EmptyCaseCondition: Avoid empty condition in case statements.
Style/EmptyElse: Avoid empty else-clauses.
Style/EmptyLambdaParameter: Omit parens for empty lambda parameters.
Style/EmptyLiteral: Prefer literals to Array.new/Hash.new/String.new.
Style/EmptyMethod: Checks the formatting of empty method definitions.
Style/Encoding: Use UTF-8 as the source file encoding.
Style/EndBlock: Avoid the use of END blocks.
Style/EvalWithLocation: Pass `__FILE__` and `__LINE__` to `eval` method, as they are used by backtraces.
Style/EvenOdd: Favor the use of `Integer#even?` && `Integer#odd?`.
Style/ExpandPathArguments: Use `expand_path(__dir__)` instead of `expand_path('..', __FILE__)`.
Style/FloatDivision: For performing float division, coerce one side only.
Style/For: Checks use of for or each in multiline loops.
Style/FormatString: Enforce the use of Kernel#sprintf, Kernel#format or String#%.
Style/FormatStringToken: Use a consistent style for format string tokens.
Style/FrozenStringLiteralComment: Add the frozen_string_literal comment to the top of files to help transition to frozen string literals by default.
Style/GlobalVars: Do not introduce global variables.
Style/GuardClause: Check for conditionals that can be replaced with guard clauses.
Style/HashSyntax: Prefer Ruby 1.9 hash syntax { a: 1, b: 2 } over 1.8 syntax { :a => 1, :b => 2 }.
Style/IdenticalConditionalBranches: Checks that conditional statements do not have an identical line at the end of each branch, which can validly be moved out of the conditional.
Style/IfInsideElse: Finds if nodes inside else, which can be converted to elsif.
Style/IfUnlessModifier: Favor modifier if/unless usage when you have a single-line body.
Style/IfUnlessModifierOfIfUnless: Avoid modifier if/unless usage on conditionals.
Style/IfWithSemicolon: Do not use if x; .... Use the ternary operator instead.
Style/ImplicitRuntimeError: Use `raise` or `fail` with an explicit exception class and message, rather than just a message.
Style/InfiniteLoop: Use Kernel#loop for infinite loops.
Style/InlineComment: Avoid trailing inline comments.
Style/InverseMethods: Use the inverse method instead of `!.method` if an inverse method is defined.
Style/IpAddresses: Don't include literal IP addresses in code.
Style/Lambda: Use the new lambda literal syntax for single-line blocks.
Style/LambdaCall: Use lambda.call(...) instead of lambda.(...).
Style/LineEndConcatenation: Use \ instead of + or << to concatenate two string literals at line end.
Style/MethodCallWithArgsParentheses: Use parentheses for method calls with arguments.
Style/MethodCallWithoutArgsParentheses: Do not use parentheses for method calls with no arguments.
Style/MethodCalledOnDoEndBlock: Avoid chaining a method call on a do...end block.
Style/MethodDefParentheses: Checks if the method definitions have or don't have parentheses.
Style/MethodMissingSuper: Checks for `method_missing` to call `super`.
Style/MinMax: Use `Enumerable#minmax` instead of `Enumerable#min` and `Enumerable#max` in conjunction.
Style/MissingElse: Require if/case expressions to have an else branches. If enabled, it is recommended that Style/UnlessElse and Style/EmptyElse be enabled. This will conflict with Style/EmptyElse if Style/EmptyElse is configured to style "both".
Style/MissingRespondToMissing: Checks if `method_missing` is implemented without implementing `respond_to_missing`.
Style/MixinGrouping: Checks for grouping of mixins in `class` and `module` bodies.
Style/MixinUsage: Checks that `include`, `extend` and `prepend` exists at the top level.
Style/ModuleFunction: Checks for usage of `extend self` in modules.
Style/MultilineBlockChain: Avoid multi-line chains of blocks.
Style/MultilineIfModifier: Only use if/unless modifiers on single line statements.
Style/MultilineIfThen: Do not use then for multi-line if/unless.
Style/MultilineMemoization: Wrap multiline memoizations in a `begin` and `end` block.
Style/MultilineMethodSignature: Avoid multi-line method signatures.
Style/MultilineTernaryOperator: Avoid multi-line ?: (the ternary operator); use if/unless instead.
Style/MultilineWhenThen: Do not use then for multi-line when statement.
Style/MultipleComparison: Avoid comparing a variable with multiple items in a conditional, use Array#include? instead.
Style/MutableConstant: Do not assign mutable objects to constants.
Style/NegatedIf: Favor unless over if for negative conditions (or control flow or).
Style/NegatedUnless: Favor if over unless for negative conditions.
Style/NegatedWhile: Favor until over while for negative conditions.
Style/NestedModifier: Avoid using nested modifiers.
Style/NestedParenthesizedCalls: Parenthesize method calls which are nested inside the argument list of another parenthesized method call.
Style/NestedTernaryOperator: Use one expression per branch in a ternary operator.
Style/Next: Use `next` to skip iteration instead of a condition at the end.
Style/NilComparison: Prefer x.nil? to x == nil.
Style/NonNilCheck: Checks for redundant nil checks.
Style/Not: Use ! instead of not.
Style/NumericLiteralPrefix: Use smallcase prefixes for numeric literals.
Style/NumericLiterals: Add underscores to large numeric literals to improve their readability.
Style/NumericPredicate: Checks for the use of predicate- or comparison methods for numeric comparisons.
Style/OneLineConditional: Favor the ternary operator(?:) over if/then/else/end constructs.
Style/OptionHash: Don't use option hashes when you can use keyword arguments.
Style/OptionalArguments: Checks for optional arguments that do not appear at the end of the argument list.
Style/OrAssignment: Recommend usage of double pipe equals (||=) where applicable.
Style/ParallelAssignment: Check for simple usages of parallel assignment. It will only warn when the number of variables matches on both sides of the assignment.
Style/ParenthesesAroundCondition: Don't use parentheses around the condition of an if/unless/while.
Style/PercentLiteralDelimiters: Use `%`-literal delimiters consistently.
Style/PercentQLiterals: Checks if uses of %Q/%q match the configured preference.
Style/PerlBackrefs: Avoid Perl-style regex back references.
Style/PreferredHashMethods: Checks use of `has_key?` and `has_value?` Hash methods.
Style/Proc: Use proc instead of Proc.new.
Style/RaiseArgs: Checks the arguments passed to raise/fail.
Style/RandomWithOffset: Prefer to use ranges when generating random numbers instead of integers with offsets.
Style/RedundantBegin: Don't use begin blocks when they are not needed.
Style/RedundantCapitalW: Checks for %W when interpolation is not needed.
Style/RedundantCondition: Checks for unnecessary conditional expressions.
Style/RedundantConditional: Don't return true/false from a conditional.
Style/RedundantException: Checks for an obsolete RuntimeException argument in raise/fail.
Style/RedundantFreeze: Checks usages of Object#freeze on immutable objects.
Style/RedundantInterpolation: Checks for strings that are just an interpolated expression.
Style/RedundantParentheses: Checks for parentheses that seem not to serve any purpose.
Style/RedundantPercentQ: Checks for %q/%Q when single quotes or double quotes would do.
Style/RedundantReturn: Don't use return where it's not required.
Style/RedundantSelf: Don't use self where it's not needed.
Style/RedundantSort: Use `min` instead of `sort.first`, `max_by` instead of `sort_by...last`, etc.
Style/RedundantSortBy: Use `sort` instead of `sort_by { |x| x }`.
Style/RegexpLiteral: Use / or %r around regular expressions.
Style/RescueModifier: Avoid using rescue in its modifier form.
Style/RescueStandardError: Avoid rescuing without specifying an error class.
Style/ReturnNil: Use return instead of return nil.
Style/SafeNavigation: This cop transforms usages of a method call safeguarded by a check for the existence of the object to safe navigation (`&.`).
Style/Sample: Use `sample` instead of `shuffle.first`, `shuffle.last`, and `shuffle[Integer]`.
Style/SelfAssignment: Checks for places where self-assignment shorthand should have been used.
Style/Semicolon: Don't use semicolons to terminate expressions.
Style/Send: Prefer `Object#__send__` or `Object#public_send` to `send`, as `send` may overlap with existing methods.
Style/SignalException: Checks for proper usage of fail and raise.
Style/SingleLineBlockParams: Enforces the names of some block params.
Style/SingleLineMethods: Avoid single-line methods.
Style/SpecialGlobalVars: Avoid Perl-style global variables.
Style/StabbyLambdaParentheses: Check for the usage of parentheses around stabby lambda arguments.
Style/StderrPuts: Use `warn` instead of `$stderr.puts`.
Style/StringHashKeys: Prefer symbols instead of strings as hash keys.
Style/StringLiterals: Checks if uses of quotes match the configured preference.
Style/StringLiteralsInInterpolation: Checks if uses of quotes inside expressions in interpolated strings match the configured preference.
Style/StringMethods: Checks if configured preferred methods are used over non-preferred.
Style/Strip: Use `strip` instead of `lstrip.rstrip`.
Style/StructInheritance: Checks for inheritance from Struct.new.
Style/SymbolArray: Use %i or %I for arrays of symbols.
Style/SymbolLiteral: Use plain symbols instead of string symbols when possible.
Style/SymbolProc: Use symbols as procs instead of blocks when possible.
Style/TernaryParentheses: Checks for use of parentheses around ternary conditions.
Style/TrailingBodyOnClass: Class body goes below class statement.
Style/TrailingBodyOnMethodDefinition: Method body goes below definition.
Style/TrailingBodyOnModule: Module body goes below module statement.
Style/TrailingCommaInArguments: Checks for trailing comma in argument lists.
Style/TrailingCommaInArrayLiteral: Checks for trailing comma in array literals.
Style/TrailingCommaInHashLiteral: Checks for trailing comma in hash literals.
Style/TrailingMethodEndStatement: Checks for trailing end statement on line of method body.
Style/TrailingUnderscoreVariable: Checks for the usage of unneeded trailing underscores at the end of parallel variable assignment.
Style/TrivialAccessors: Prefer attr_* methods to trivial readers/writers.
Style/UnlessElse: Do not use unless with else. Rewrite these with the positive case first.
Style/UnpackFirst: Checks for accessing the first element of `String#unpack` instead of using `unpack1`.
Style/VariableInterpolation: Don't interpolate global, instance and class variables directly in strings.
Style/WhenThen: Use when x then ... for one-line cases.
Style/WhileUntilDo: Checks for redundant do after while or until.
Style/WhileUntilModifier: Favor modifier while/until usage when you have a single-line body.
Style/WordArray: Use %w or %W for arrays of words.
Style/YodaCondition: Forbid or enforce yoda conditions.
Style/ZeroLengthPredicate: Use #empty? when testing for objects of length 0.
