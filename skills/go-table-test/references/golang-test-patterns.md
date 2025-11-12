# Golang Table Driven Test Patterns

## Core Structure

```go
func TestFunctionName(t *testing.T) {
    tests := []struct {
        name    string
        // Input fields
        input   InputType
        // Setup fields (for mocks, DB, etc.)
        setup   func(*testing.T) (cleanup func())
        // Expected output
        want    OutputType
        wantErr bool
    }{
        {
            name: "success case: description",
            input: InputType{...},
            setup: func(t *testing.T) func() {
                // Setup mocks or DB
                return func() {
                    // Cleanup
                }
            },
            want: OutputType{...},
            wantErr: false,
        },
        {
            name: "error case: description",
            input: InputType{...},
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

            // When: Execute
            got, err := FunctionName(tt.input)

            // Then: Assert
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

## Pattern 1: Simple Function Test

For pure functions without dependencies:

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name string
        a    int
        b    int
        want int
    }{
        {
            name: "positive numbers",
            a:    1,
            b:    2,
            want: 3,
        },
        {
            name: "negative numbers",
            a:    -1,
            b:    -2,
            want: -3,
        },
        {
            name: "zero",
            a:    0,
            b:    0,
            want: 0,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: inputs are prepared

            // When
            got := Add(tt.a, tt.b)

            // Then
            assert.Equal(t, tt.want, got)
        })
    }
}
```

## Pattern 2: With Mock Dependencies

Using testify/mock:

```go
func TestServiceMethod(t *testing.T) {
    tests := []struct {
        name      string
        input     ServiceInput
        setupMock func(*MockRepository)
        want      ServiceOutput
        wantErr   bool
    }{
        {
            name: "success: repository returns data",
            input: ServiceInput{ID: 1},
            setupMock: func(m *MockRepository) {
                m.On("Get", 1).Return(&Data{ID: 1, Name: "test"}, nil)
            },
            want:    ServiceOutput{Name: "test"},
            wantErr: false,
        },
        {
            name: "error: repository returns error",
            input: ServiceInput{ID: 1},
            setupMock: func(m *MockRepository) {
                m.On("Get", 1).Return(nil, errors.New("not found"))
            },
            want:    ServiceOutput{},
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: Setup mock
            mockRepo := new(MockRepository)
            if tt.setupMock != nil {
                tt.setupMock(mockRepo)
            }
            defer mockRepo.AssertExpectations(t)

            svc := NewService(mockRepo)

            // When
            got, err := svc.Method(tt.input)

            // Then
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

## Pattern 3: With Database Setup

For integration tests with real DB:

```go
func TestRepositoryCreate(t *testing.T) {
    tests := []struct {
        name    string
        input   *User
        setup   func(*testing.T, *sql.DB)
        want    *User
        wantErr bool
    }{
        {
            name: "success: create new user",
            input: &User{Name: "John", Email: "john@example.com"},
            setup: func(t *testing.T, db *sql.DB) {
                // Clean up existing data
                _, err := db.Exec("DELETE FROM users WHERE email = ?", "john@example.com")
                require.NoError(t, err)
            },
            want: &User{ID: 1, Name: "John", Email: "john@example.com"},
            wantErr: false,
        },
        {
            name: "error: duplicate email",
            input: &User{Name: "Jane", Email: "duplicate@example.com"},
            setup: func(t *testing.T, db *sql.DB) {
                // Insert existing user
                _, err := db.Exec("INSERT INTO users (name, email) VALUES (?, ?)", "Existing", "duplicate@example.com")
                require.NoError(t, err)
            },
            want:    nil,
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: Setup database
            db := setupTestDB(t)
            defer cleanupTestDB(t, db)

            if tt.setup != nil {
                tt.setup(t, db)
            }

            repo := NewRepository(db)

            // When
            got, err := repo.Create(tt.input)

            // Then
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.want.Name, got.Name)
            assert.Equal(t, tt.want.Email, got.Email)
        })
    }
}
```

## Pattern 4: Complex Assertions

When each test case needs different assertion logic:

```go
func TestComplexValidation(t *testing.T) {
    tests := []struct {
        name      string
        input     ValidationInput
        want      ValidationOutput
        wantErr   bool
        assertFn  func(*testing.T, ValidationOutput, error)
    }{
        {
            name:  "success: all fields valid",
            input: ValidationInput{Name: "John", Age: 30, Email: "john@example.com"},
            assertFn: func(t *testing.T, got ValidationOutput, err error) {
                assert.NoError(t, err)
                assert.True(t, got.Valid)
                assert.Empty(t, got.Errors)
            },
        },
        {
            name:  "error: multiple validation errors",
            input: ValidationInput{Name: "", Age: -1, Email: "invalid"},
            assertFn: func(t *testing.T, got ValidationOutput, err error) {
                assert.NoError(t, err) // Function doesn't return error, but returns validation result
                assert.False(t, got.Valid)
                assert.Len(t, got.Errors, 3)
                assert.Contains(t, got.Errors, "name is required")
                assert.Contains(t, got.Errors, "age must be positive")
                assert.Contains(t, got.Errors, "email is invalid")
            },
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: input is prepared

            // When
            got, err := Validate(tt.input)

            // Then
            if tt.assertFn != nil {
                tt.assertFn(t, got, err)
            }
        })
    }
}
```

## Pattern 5: Context-Based Functions

For functions that accept context.Context:

```go
func TestWithContext(t *testing.T) {
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
            input:   Input{ID: 1},
            want:    Output{Data: "success"},
            wantErr: false,
        },
        {
            name: "error: context timeout",
            ctx: func() context.Context {
                ctx, cancel := context.WithTimeout(context.Background(), 1*time.Nanosecond)
                defer cancel()
                time.Sleep(10 * time.Millisecond) // Ensure timeout
                return ctx
            },
            input:   Input{ID: 1},
            want:    Output{},
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given
            ctx := tt.ctx()

            // When
            got, err := FunctionWithContext(ctx, tt.input)

            // Then
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

## Pattern 6: Pointer vs Value Comparison

For comparing complex structs:

```go
func TestStructComparison(t *testing.T) {
    tests := []struct {
        name    string
        input   Input
        want    *Output
        wantErr bool
    }{
        {
            name: "success: returns populated struct",
            input: Input{ID: 1},
            want: &Output{
                ID:   1,
                Name: "test",
                CreatedAt: time.Date(2024, 1, 1, 0, 0, 0, 0, time.UTC),
            },
            wantErr: false,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: input prepared

            // When
            got, err := GetOutput(tt.input)

            // Then
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)

            // Use assert.Equal for simple comparison
            assert.Equal(t, tt.want, got)

            // Or use field-by-field comparison for more control
            assert.Equal(t, tt.want.ID, got.ID)
            assert.Equal(t, tt.want.Name, got.Name)
            assert.WithinDuration(t, tt.want.CreatedAt, got.CreatedAt, time.Second)
        })
    }
}
```

## Best Practices

### 1. Test Naming Convention

- Use descriptive test case names: `"success: specific scenario"` or `"error: specific failure"`
- Format: `"[success|error]: description of scenario"`

### 2. Given-When-Then Comments

Always include these three sections for clarity:

```go
// Given: Setup and preconditions
// When: Action being tested
// Then: Assertions
```

### 3. Error Handling

Always check `wantErr` first and return early:

```go
if tt.wantErr {
    assert.Error(t, err)
    return
}
assert.NoError(t, err)
// Continue with success assertions
```

### 4. Cleanup

Use `defer` for cleanup functions:

```go
if tt.setup != nil {
    cleanup := tt.setup(t)
    defer cleanup()
}
```

### 5. Mock Assertions

Always assert mock expectations:

```go
defer mockRepo.AssertExpectations(t)
```

### 6. Table Field Ordering

Standard field order in test struct:
1. `name` (always first)
2. Input fields
3. Setup/mock functions
4. Expected output (`want`, `wantErr`)
5. Custom assertion functions (if needed)

### 7. Context Usage

For tests using context.Background():

```go
// Use t.Context() instead of context.Background()
ctx := t.Context()
```

This ensures proper cleanup and timeout handling in tests.

### 8. Require vs Assert

- Use `require.*` for setup/preconditions (stops test on failure)
- Use `assert.*` for actual test assertions (continues to show all failures)

```go
// Setup - use require (fail fast if setup fails)
require.NoError(t, setupErr)

// Assertions - use assert (show all failures)
assert.Equal(t, expected, actual)
assert.True(t, condition)
```
