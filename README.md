## uploadcare-go

Go library for accessing Uploadcard API https://uploadcare.com/

### Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Usage examples](#usage-examples)

### Requirements

go1.13

### Installation

To install the library simply run:

```
go get -u -v github.com/uploadcare/uploadcare-go/...
```

### Usage examples

Getting a paginated list of files:

```go
import(
	"github.com/uploadcare/uploadcare-go/uploadcare"
	"github.com/uploadcare/uploadcare-go/file"
	...
)

func main() {
	// your project credentials
	creds := uploadcare.APICreds{
		SecretKey: "your-project-secret-key",
		PublicKey: "your-project-public-key",
	}

	// creating underlying client which is responsible for
	// actual API calls and authentication
	client, err := uploadcare.NewClient(
		creds,
		WithAuthentication(uploadcare.WithSignBasedAuth),
	)
	if err != nil {
		log.Fatal("creating uploadcare API client: %s", err)
	}

	// creating a service that performs file operations 
	fileSvc := file.New(client) 

	listParams := &file.ListFilesParams{
		Stored: uploadcare.String(true),
		Ordering: uploadcare.String(file.OrderBySizeAsc),
	}
	
	fileList, err := fileSvc.ListFiles(context.Background(), listParams)
	if err != nil {
		// handle error
	}
			
	// getting file IDs for the first 100 files (ordered by size)
	ids := make([]string, 0, 100)
	for fileList.HasNext() && fileList.Count() <= 100 {
		id := fileList.Next().ID
		ids = append(ids, id)
	}

	...
}
```

Acquiring file-specific info:

```go
	fileID := ids[0]
	file, err := fileSvc.FileInfo(context.Background(), fileID)
	if err != nil {
		// handle error
	}

	if file.IsImage {
		h := file.ImageInfo.Height
		w := file.ImageInfo.Width
		fmt.Printf("image size: %dx%d\n", h, w)
	}
```

Deleting a file:

```go
	_, err := fileSvc.DeleteFile(context.Background(), fileID)
	if err != nil {
		// handle error
	}
```

Storing a single file by ID:

```go
	_, err := fileSvc.StoreFile(context.Background(), fileID)
	if err != nil {
		// handle error
	}
```

Getting a list of groups:

```go
	groupSvc := group.New(client)
	
	groupList, err := groupSvc.ListGroups()
	if err != nil {
		// handle error
	}

	// getting group IDs
	groupIDs = make([]string, 0, groupList.Total)
	for groupList.HasNext() {
		groupIDs = append(groupIDs, groupList.Next().ID)
	}

```

Getting a file group by ID:

```go
	groupID := groupIDs[0]
	group, err := groupSvc.GroupInfo(context.Background(), groupID)
	if err != nil {
		// handle error
	}

	fmt.Printf("group %s contains %d files\n", group.ID, group.FileCount)

```

Marking all files in a group as stored:

```go
	_, err := groupSvc.StoreGroup(context.Background(), groupID)
	if err != nil {
		// handle error
	}
```
