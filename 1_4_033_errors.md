# Real-World Error Example: AV Repo

An oversight in our AV repo led to a production issue:

```go
func processEvent(event events.Event) {
	helpers.GetForwardManager().EventStream <- event
}
```

**Problem:**  
This code assumes that both `GetForwardManager()` and its `EventStream` property are never `nil`. If either is `nil`, this will panic at runtime, causing the application to crash.

**Improved Version with Error Handling:**

```go
func processEvent(event events.Event) {
	//helpers.GetForwardManager().EventStream <- event
	forwardManager := helpers.GetForwardManager()

	if forwardManager == nil {
		slog.Error("GetForwardManager() returned nil!")
		return
	}

	if forwardManager.EventStream == nil {
		slog.Error("ForwardManager.EventStream is nil! Initialization might have failed.")
		return
	}

	slog.Debug("processEvent() received event:", "event", event)
	forwardManager.EventStream <- event
}
```

**Why is this beneficial?**  
- Prevents panics and application crashes by checking for `nil` before use.
- Provides clear error logs, making debugging easier.
- Ensures the system can continue running even if a component fails to initialize.

---


## When to Log vs. When to Pass Errors

- **Log the error:**  
  - At the top level of your application (e.g., HTTP handler, main loop).
  - When you have enough context to provide a meaningful message.
  - When the error cannot or should not be handled further up.

- **Pass the error up:**  
  - When a lower-level function/package cannot handle the error meaningfully.
  - When you want the caller to decide how to handle or log the error.
  - Use `return err` or wrap the error with additional context.

**Rule of thumb:**  
- Log errors once, at the highest level where you have enough context.
- Pass errors up from libraries and packages; let the application decide how to handle them.

---

## Sanitizing and Returning Errors

- **Always sanitize errors before returning them to APIs or clients.**  
  Never expose internal details, stack traces, or sensitive information in error messages sent outside your service.
- **Recommendation:**  
  - Log the full error internally for debugging.
  - Pass a sanitized, user-friendly error message to the API response or client.
  - Review all error paths to ensure no sensitive data is leaked.

Example:

```go
if err != nil {
	slog.Error("failed to process event", "err", err)
	return errors.New("unable to process request at this time")
}
```

---

## Logging Best Practices with slog

- **Use standardized key-value pairs when logging errors.**
  - This ensures consistency and makes logs easier to search, filter, and analyze.
  - Common keys: `"err"`, `"event"`, `"user_id"`, `"request_id"`, etc.
  - Example:
    ```go
    slog.Error("failed to process event", "err", err, "event_id", event.ID)
    ```
- **Recommendation:**  
  - Define and document a set of standard keys for your team or project.
  - Always include relevant context (such as IDs or operation names) in your logs.

