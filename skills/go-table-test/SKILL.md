---
name: go-table-test
description: Generate table-driven tests for Golang functions and methods using Given-When-Then structure. This skill should be used when asked to write tests for Go functions, methods, or files. It creates or updates *_test.go files with comprehensive table-driven tests that follow best practices including mock setup, database preparation, and complex assertions.
---

# Go Table Test

## Overview

Generate comprehensive table-driven tests for Golang functions using the Given-When-Then pattern. Create or update `*_test.go` files with well-structured tests that include input setup, mock configuration, database preparation, and proper assertions using testify.

## When to Use This Skill

Use this skill when:
- Asked to "write tests for [function/method/file]"
- Need to create table-driven tests for Go code
- Adding test coverage to existing Go functions
- Creating test files for new Go implementations

## Core Workflow

### 1. Identify Target Functions

When asked to write tests:

**For a specific function:**
- Read the function implementation
- Identify function signature, inputs, outputs, and error conditions

**For an entire file:**
- List all exported functions in the file
- Generate tests for each function
- Create or append to corresponding `*_test.go` file

**For a package/directory:**
- Find all `.go` files (excluding `*_test.go`)
- Generate tests for exported functions in each file

### 2. Determine Test File Location

**File naming convention:**
- If testing `service.go`, use `service_test.go`
- If `service_test.go` exists, append tests to it
- If it doesn't exist, create a new file

**Package declaration:**
- Use the same package as the source file
- Example: `package service` (not `package service_test`)

### 3. Analyze Function Requirements

For each function, identify:

**Inputs:**
- Function parameters
- Context (if using `context.Context`)
- Dependencies (interfaces, repositories, services)

**Outputs:**
- Return values
- Error conditions

**Dependencies:**
- Which dependencies need mocking?
- Which need database setup?
- Which require context configuration?

**Test Cases:**
- Success scenarios (1-3 cases)
- Error scenarios (1-3 cases)
- Edge cases (zero values, nil pointers, etc.)

### 4. Structure Test Table

Use this standard structure:

```go
func TestFunctionName(t *testing.T) {
    tests := []struct {
        name      string
        // Input fields
        input     InputType
        // Setup functions for mocks/DB
        setup     func(*testing.T) (cleanup func())
        setupMock func(*MockDependency)
        // Expected output
        want      OutputType
        wantErr   bool
        // Optional: custom assertion
        assertFn  func(*testing.T, OutputType, error)
    }{
        {
            name: "success: description",
            input: InputType{...},
            setupMock: func(m *MockDependency) {
                m.On("Method", args).Return(result, nil)
            },
            want: OutputType{...},
            wantErr: false,
        },
        {
            name: "error: description",
            input: InputType{...},
            setupMock: func(m *MockDependency) {
                m.On("Method", args).Return(nil, errors.New("error"))
            },
            want: OutputType{},
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: Setup
            if tt.setup != nil {
                cleanup := tt.setup(t)
                defer cleanup()
            }

            if tt.setupMock != nil {
                mockDep := new(MockDependency)
                tt.setupMock(mockDep)
                defer mockDep.AssertExpectations(t)
                // Use mockDep in function call
            }

            // When: Execute
            got, err := FunctionName(tt.input)

            // Then: Assert
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)

            if tt.assertFn != nil {
                tt.assertFn(t, got, err)
            } else {
                assert.Equal(t, tt.want, got)
            }
        })
    }
}
```

### 5. Handle Different Function Types

**Pure functions (no dependencies):**
```go
tests := []struct {
    name string
    a    int
    b    int
    want int
}{
    {name: "success: positive numbers", a: 1, b: 2, want: 3},
    {name: "success: negative numbers", a: -1, b: -2, want: -3},
}
```

**Functions with interface dependencies:**
```go
tests := []struct {
    name      string
    input     Input
    setupMock func(*MockRepository)
    want      Output
    wantErr   bool
}{
    {
        name: "success: repository returns data",
        input: Input{ID: 1},
        setupMock: func(m *MockRepository) {
            m.On("Get", 1).Return(&Data{}, nil)
        },
        want: Output{},
        wantErr: false,
    },
}
```

**Functions requiring database setup:**
```go
tests := []struct {
    name    string
    input   Input
    setup   func(*testing.T, *sql.DB)
    want    Output
    wantErr bool
}{
    {
        name: "success: data exists",
        input: Input{ID: 1},
        setup: func(t *testing.T, db *sql.DB) {
            _, err := db.Exec("INSERT INTO table VALUES (?)", 1)
            require.NoError(t, err)
        },
        want: Output{},
        wantErr: false,
    },
}
```

**Functions with context:**
```go
tests := []struct {
    name    string
    ctx     func() context.Context
    input   Input
    want    Output
    wantErr bool
}{
    {
        name: "success: normal context",
        ctx: func() context.Context {
            return context.Background()
        },
        input: Input{},
        want: Output{},
        wantErr: false,
    },
    {
        name: "error: context canceled",
        ctx: func() context.Context {
            ctx, cancel := context.WithCancel(context.Background())
            cancel()
            return ctx
        },
        input: Input{},
        want: Output{},
        wantErr: true,
    },
}
```

**Functions with complex assertions:**
```go
tests := []struct {
    name     string
    input    Input
    assertFn func(*testing.T, Output, error)
}{
    {
        name: "success: multiple validations",
        input: Input{},
        assertFn: func(t *testing.T, got Output, err error) {
            assert.NoError(t, err)
            assert.NotEmpty(t, got.ID)
            assert.True(t, got.IsValid)
            assert.Len(t, got.Items, 3)
        },
    },
}
```

### 6. Import Required Packages

Always include at the top of test file:

```go
import (
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/stretchr/testify/mock"
)
```

Add additional imports as needed:
- `"context"` - for context-based tests
- `"errors"` - for error creation
- `"time"` - for time-based tests
- Package imports for types being tested

### 7. Generate Mock Types

If function has interface dependencies, generate mocks using testify/mock:

```go
// MockRepository is a mock implementation of Repository interface
type MockRepository struct {
    mock.Mock
}

func (m *MockRepository) Get(id int) (*Data, error) {
    args := m.Called(id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*Data), args.Error(1)
}

func (m *MockRepository) Create(data *Data) error {
    args := m.Called(data)
    return args.Error(0)
}
```

Place mock definitions at the bottom of the test file.

## Best Practices

### Test Case Naming

Use descriptive names with prefixes:
- `"success: specific scenario"` - for happy path tests
- `"error: specific failure"` - for error cases
- `"edge case: specific condition"` - for edge cases

Example:
```go
{name: "success: user exists and is active", ...},
{name: "error: user not found", ...},
{name: "edge case: empty input", ...},
```

### Given-When-Then Structure

Always include these three sections as comments:

```go
t.Run(tt.name, func(t *testing.T) {
    // Given: Setup preconditions and mocks

    // When: Execute the function under test

    // Then: Assert the results
})
```

### Error Handling Pattern

Always check `wantErr` first and return early:

```go
// Then
if tt.wantErr {
    assert.Error(t, err)
    return
}
assert.NoError(t, err)
// Continue with success assertions
```

### Context Best Practice

Use `t.Context()` instead of `context.Background()` in tests:

```go
// Use this
ctx := t.Context()

// Not this
ctx := context.Background()
```

This ensures proper cleanup and timeout handling.

### Cleanup Pattern

Use defer for cleanup functions:

```go
if tt.setup != nil {
    cleanup := tt.setup(t)
    defer cleanup()
}
```

### Mock Assertions

Always assert mock expectations:

```go
defer mockRepo.AssertExpectations(t)
```

### Require vs Assert

- Use `require.*` for setup/preconditions (stops test on failure)
- Use `assert.*` for actual test assertions (continues to show all failures)

```go
// Setup - use require
require.NoError(t, setupErr)

// Assertions - use assert
assert.Equal(t, expected, actual)
```

## Common Patterns Reference

For detailed examples of common testing patterns, refer to:
- `references/golang-test-patterns.md` - comprehensive pattern examples

Read this file when encountering:
- Complex mock setups
- Database integration tests
- Context-based testing
- Custom assertion logic
- Pointer vs value comparisons

## Output Format

Generate complete, runnable test code that:
1. Includes all necessary imports
2. Has proper package declaration
3. Contains comprehensive test cases (success, error, edge cases)
4. Uses Given-When-Then structure
5. Includes mock definitions if needed
6. Follows Go testing best practices
7. Uses testify assertions

## Example Usage Scenarios

**Scenario 1: "Write tests for the CreateUser function"**
1. Read the CreateUser function implementation
2. Identify inputs (user data), outputs (created user, error), dependencies (UserRepository)
3. Generate mock for UserRepository
4. Create test cases: success, duplicate email error, validation error
5. Write table-driven test with Given-When-Then structure
6. Save to appropriate `*_test.go` file

**Scenario 2: "Add tests to service.go"**
1. Read service.go to find all exported functions
2. Check if service_test.go exists
3. For each function, generate table-driven tests
4. Append tests to service_test.go (or create new file)

**Scenario 3: "Write tests for the entire auth package"**
1. Find all .go files in auth package
2. For each file, identify exported functions
3. Generate comprehensive test coverage
4. Create corresponding *_test.go files
