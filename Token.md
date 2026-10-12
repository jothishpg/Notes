Base64.getUrlEncoder().withoutPadding()
withoutPadding() means remove the = characters that Base64 normally adds at the end

Base64 Encoding
Three bytes become four characters.

Plain Text
32 bytes
↓
about
Plain Text
44 characters

Hex Encoding
Each byte becomes two characters.

Plain Text
32 bytes
↓
Plain Text
64 characters
