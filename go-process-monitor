package main

import (
    "fmt"
    "os"
    "syscall"
    "github.com/Microsoft/go-winio"
)

func main() {
    // Parse command line arguments
    if len(os.Args) < 2 {
        fmt.Println("Usage: go-process-monitor [process_id|process_name]")
        os.Exit(1)
    }
    pidOrName := os.Args[1]

    // Open the Process Monitor Live driver
    h, err := winio.CreateFile(&winio.SecurityAttributes{}, syscall.FILE_READ_ATTRIBUTES, syscall.FILE_SHARE_READ|syscall.FILE_SHARE_WRITE, winio.OPEN_EXISTING, winio.FILE_FLAG_OVERLAPPED|winio.FILE_FLAG_NO_BUFFERING, 0)
    if err != nil {
        fmt.Println("Failed to open Process Monitor Live driver:", err)
        os.Exit(1)
    }
    defer h.Close()

    // Create an overlapped I/O structure
    o := &winio.Overlapped{}

    // Set the filter to include only the specified process id or process name
    filter := winio.Filter{
        ProcessId: pidOrName,
        Include:   winio.FilterIncludeAll,
    }

    // Set the buffer size for the events
    bufSize := uint32(64 * 1024)

    // Set the output mode to include detailed information for each event
    mode := uint32(winio.OutputModeDetailed)

    // Apply the filter and output mode to the Process Monitor Live driver
    err = winio.SetFilter(h, &filter, mode, bufSize)
    if err != nil {
        fmt.Println("Failed to set filter and output mode:", err)
        os.Exit(1)
    }

    // Loop forever to read the events from the driver
    for {
        // Allocate a buffer to hold the events
        buf := make([]byte, bufSize)

        // Read the events from the driver
        n, err := winio.Read(h, buf, o)
        if err != nil {
            fmt.Println("Failed to read events:", err)
            os.Exit(1)
        }

        // Parse the events from the buffer
        events, err := winio.ParseEvents(buf[:n], mode)
        if err != nil {
            fmt.Println("Failed to parse events:", err)
            os.Exit(1)
        }

        // Output the events to the console
        for _, e := range events {
            fmt.Println(e)
        }
    }
}
