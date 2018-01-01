# Hydra Data Model (HDM)

As specified by HDM, a post in ZPL:
```
post
    ident = "DCCCECB20793342E3BAC617020D8F821037A80F7"
    subject = "contains contents of 1024 size"
    timestamp = "2018-01-01T19:08:18Z"
    parent-id = ""
    mime-type = "*/*"
    digest = "60CACBF3D72E1E7834203DA608037B1BF83B40E8"
    location = "posts/blobs/60CACBF3D72E1E7834203DA608037B1BF83B40E8"
    content-size = "1024"
```

As defined by the Hydra Protocol ABNF grammar, a post on the wire:
```
;  Server returns the metadata for the current post (as returned by      
;  NEXT-OK).                                                             
META-OK         = signature %d8 subject timestamp parent_id digest mime_type content_size
subject         = longstr               ; Subject line
timestamp       = string                ; Post creation timestamp
parent id       = string                ; Parent post ID, if any
digest          = string                ; Content SHA1 digest
mime type       = string                ; Content MIME type
content size    = number-8              ; Content size, octets
```
Content chunks are requested/received after this META-OK(see Hydra readme)

## Potential Extensions to Hydra Data Model:

### Tag/Pair (Proposal):
A Tag is a single value consisting of a string.

A Pair is a key:value pair with key as string and value
as string to follow ZPL or xml formatting.

A post may have zero or more tags or pairs.

A post content block may have zero or more tags or pairs.

A content block refers to the collection of necessary metadata for a piece of content: digest, mime type, content size (and potentially tag/pair)

### Tag/Pair Scope:

* Post-level: tags and pairs associated to the complete post. Allows higher-level grouping and metadata.

* Content-level: tags and pairs associated with specific content. Allows metadata such as page numbers per image or other reference systems.   

### Multipart Content(Proposal):

To allow multipart content, provide structure to support multiple content blocks in a post.  

ZPL:

Using ZPL content could be enumerated using a prefix-integer enumeration scheme(ie content-1,content-2...) to avoid overwriting keys. ZPL is lightweight and already used by Hydra

XML: 

Using XML format provides easy support for multiple content nodes and outputs using gsl but adds parsing complexity

Proposal: Use ZPL and if needed write a conversion script/function for ZPL to XML (which can be then consumed by gsl). 

Implementation:

* wire framing for content blocks and post tag/pair?

* Requesting / fetching and checking multiple content blobs would have to be added

* Moving to more sophisticated chunking of content is still TODO. See `prepare_to_get_first_chunk` in `hydra_client.c`

### HDM Version identifier(Speculative):

Add `version` field to post to help parsing and compatibility

version = "HDMP01"

Hydra Data Model Post 0.1

## Examples:

* Potential ZPL post with multipart content, tag/pair and scoped tag/pair
    ```
    post
        ident = "DCCCECB20793342E3BAC617020D8F821037A80F7"
        subject = "contains contents of 1024 size"
        timestamp = "2018-01-01T19:08:18Z"
        parent-id = ""
        tag
            post_level_tag1
            post_level_tag2
        pair
            some_key = some_value
            some_other_key = some_data
        content-1
            mime-type = "*/*"
            digest = "60CACBF3D72E1E7834203DA608037B1BF83B40E8"
            location = "posts/blobs/60CACBF3D72E1E7834203DA608037B1BF83B40E8"
            content-size = "1024"
            tag
                something
                something_else
            pair
                some_metadata = some_value
        content-2
            ...
    ```

* XML of current ZPL post following HDM
    ```
    <!-- xml, exact copy of ZPL -->
    <post ident = "DCCCECB20793342E3BAC617020D8F821037A80F7"
          subject = "contains contents of 1024 size"
          timestamp = "2018-01-01T19:08:18Z"
          parent-id = ""
          mime-type = "*/*"
          digest = "60CACBF3D72E1E7834203DA608037B1BF83B40E8"
          location = "posts/blobs/60CACBF3D72E1E7834203DA608037B1BF83B40E8"
          content-size = "1024"
    />
```

* Potential XML post with multipart content, tag/pair and scoped tag/pair
```
<!-- xml with support for multipart content -->
<post ident = "DCCCECB20793342E3BAC617020D8F821037A80F7"
      subject = "contains contents of 1024 size"
      timestamp = "2018-01-01T19:08:18Z"
      parent-id = "">

    <tag value="post_level_tag" />
    <pair key="post_level_metadata" value="some value" />

    <!-- content without tags or pairs -->
    <content digest = "60CACBF3D72E1E7834203DA608037B1BF83B40E8"
             mime-type = "*/*"  
             location = "posts/blobs/60CACBF3D72E1E7834203DA608037B1BF83B40E8"
             content-size = "1024" />

    <!-- content with scoped tag and pair -->
    <content digest = "60CACBF3D72E1E7834203DA608037B1BF83B40E8" 
             mime-type = "*/*"  
             location = "posts/blobs/60CACBF3D72E1E7834203DA608037B1BF83B40E8"
             content-size = "1024">
        <tag value="tag_for_this_content" />
        <pair key="page" value="4" />
        <pair key="timestamp" value="2017-01-01T19:08:18Z" />
    </content>
</post>
```