---
UID: NF:dbgmodel.IDebugHostPublic.EnumerateChildren
title: IDebugHostPublic::EnumerateChildren (dbgmodel.h)
description: "The IDebugHostPublic::EnumerateChildren method gets an enumerator capable of enumerating all children of a given symbol."
ms.date: 09/21/2018
keywords: ["IDebugHostPublic::EnumerateChildren"]
ms.keywords: IDebugHostPublic::EnumerateChildren, EnumerateChildren, IDebugHostPublic.EnumerateChildren, IDebugHostPublic::EnumerateChildren, IDebugHostPublic.EnumerateChildren
req.header: dbgmodel.h
req.include-header: 
req.target-type: 
req.target-min-winverclnt: 
req.target-min-winversvr: 
req.kmdf-ver: 
req.umdf-ver: 
req.lib: 
req.dll: 
req.irql: 
req.ddi-compliance: 
req.unicode-ansi: 
req.idl: 
req.max-support: 
req.namespace: 
req.assembly: 
req.type-library: 
targetos: Windows
tech.root: debugger
ms.custom: RS5
f1_keywords:
 - IDebugHostPublic::EnumerateChildren
 - dbgmodel/IDebugHostPublic::EnumerateChildren
topic_type:
 - apiref
api_type:
 - COM
api_location:
 - dbgmodel.h
api_name:
 - IDebugHostPublic::EnumerateChildren
---

# IDebugHostPublic::EnumerateChildren


## -description

The EnumerateChildren method returns an enumerator which will enumerate all children of a given symbol. For a C++ type, for example, the base classes, fields, member functions, and the like are all considered children of the type symbol.

## -parameters

### -param kind

Indicates what kinds of child symbols the caller wishes to enumerate. If the flat value Symbol is passed, all kinds of child symbols will be enumerated.

### -param name

If specified, only child symbols with a name as given in this argument will be enumerated.

### -param ppEnum

An enumerator which enumerates child symbols of the specified kind and name will be returned here.

## -returns

This method returns HRESULT that indicates success or failure.

## -remarks

**Code Sample**

```cpp
ComPtr<IDebugHostType> spType; /* get the type of an object */

// Enumerate every field of this type.  Note that this *WILL NOT* enumerate 
// fields of base classes!
ComPtr<IDebugHostSymbolEnumerator> spEnum;
if (SUCCEEDED(spType->EnumerateChildren(SymbolField, nullptr, &spEnum)))
{
    ComPtr<IDebugHostSymbol> spFieldSymbol;
    HRESULT hr = S_OK;
    while (SUCCEEDED(hr))
    {
        hr = spEnum->GetNext(&spFieldSymbol);
        if (SUCCEEDED(hr))
        {
            ComPtr<IDebugHostField> spField;
            if (SUCCEEDED(spFieldSymbol.As(&spField))) /* should always succeed */
            {
                // spField is each field of the type in turn
            }
        }
    }

    // hr == E_BOUNDS : we hit the end of the enumerator
    // hr == E_ABORT  : user requested interruption, propagate upwards immediately
}
```

## -see-also

[IDebugHostPublic interface](nn-dbgmodel-idebughostpublic.md)

