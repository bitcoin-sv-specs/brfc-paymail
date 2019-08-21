---
brfc: true
title: Public Profile (Name & Avatar)
authors:
  - Ryan X. Charles (Money Button)
version: 1
---
# Name & Avatar

{{yfm}}

This capability allows the display of a name and avatar for a given paymail, with the possibility to extend the available information in the future.

# Motivation

It would be nice if we had extensible protocol to retrieve public profile information, particularly a name and avatar, for a given paymail. Then these items can be displayed next to a paymail inside of a contact list or any other UI component.

The name should be a string sufficiently long as to allow any human name, business name, or entity name.

The avatar should be a URL to an image using an API similar to Gravatar.

Name and avatar are the most common, but it would be good if the protocol were extensible so that services could add other pieces of public profile information (such as website) if such pieces of information were deemed important. The protocol is naturally extensible simply by using JSON.

## Capability discovery

The `.well-known/bsvalias` document is updated to include a public profile endpoint:

```json
{
  "bsvalias": "1.0",
  "capabilities": {
    "{{fm:brfc}}": "https://example.bsvalias.tld/api/{alias}@{domain.tld}/contact-card"
  }
}
```

The `capabilities.{{fm:brfc}}` is a template URL to query for the public profile information.

## Client Request

The `capabilities.{{fm:brfc}}` path returns a URI template. Senders _MUST_ replace `{alias}`, `{domain.tld}` placeholders with a valid paymail handle.

The client _MUST_ perform a GET request to the obtained URL.

## Server Response

The server response is a JSON object containing public profile information. The only pieces of information are name and avatar. However, because we use JSON, it will be possible to extend this protocol to include additional information later on.

### 200 OK

Returned when a valid request was made. The response _MUST_ have `application/json` as content type. The response body _MUST_ follow this schema:

```json
{
  "name":"[a person'a name, a business name, or any other name associated with the paymail]",
  "avatar":"https://example.com/api/image.png{?s}"
}
```

| Field    | Description                                                                                                           |
|----------|-----------------------------------------------------------------------------------------------------------------------|
| `name`   | A string up to 100 characters long.                                                                                   |
| `avatar` | A URL that returns a 80x80 image. It can accept an optional parameter `s` to return an image of width and height `s`. |

The image should be JPEG, PNG, or GIF.

The returned image _MUST_ be a square. The returned image _SHOULD_ be at the resolution specified by `s`. The maximum size of `s` _MUST NOT_ exceed 1000 pixels. In addition, the image _SHOULD_ look good when cropped to a circle, since this will be the most common way to display the images.

If the returned image is the wrong size, the viewer _MUST NOT_ display the image, and _MUST_ display a placeholder image instead. This will enforce that all returned images are square.
