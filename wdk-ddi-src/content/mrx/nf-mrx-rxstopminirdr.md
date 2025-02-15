---
UID: NF:mrx.RxStopMinirdr
title: RxStopMinirdr function (mrx.h)
description: Learn more about the RxStopMinirdr function.
tech.root: ifsk
ms.date: 10/11/2023
keywords: ["RxStopMinirdr function"]
ms.keywords: RxStopMinirdr, RxStopMinirdr function [Installable File System Drivers], ifsk.rxstopminirdr, mrx/RxStopMinirdr, rxref_3ec6b643-e13c-4aa6-879b-fdb2076e549d.xml
req.header: mrx.h
req.include-header: Mrx.h
req.target-type: Desktop
req.target-min-winverclnt: 
req.target-min-winversvr: 
req.kmdf-ver: 
req.umdf-ver: 
req.ddi-compliance: 
req.unicode-ansi: 
req.idl: 
req.max-support: 
req.namespace: 
req.assembly: 
req.type-library: 
req.lib: 
req.dll: 
req.irql: <= APC_LEVEL
targetos: Windows
req.typenames: 
f1_keywords:
 - RxStopMinirdr
 - mrx/RxStopMinirdr
topic_type:
 - APIRef
 - kbSyntax
api_type:
 - HeaderDef
api_location:
 - mrx.h
api_name:
 - RxStopMinirdr
---

# RxStopMinirdr function

## -description

**RxStopMinirdr** is called to stop a network mini-redirector that has previously been started. As part of **RxStopMinirdr**, RDBSS will also de-register the network mini-redirector driver as a universal naming convention (UNC) provider with the Multiple UNC Provider (MUP) if the driver indicates support for UNC names.

## -parameters

### -param RxContext [in]

A pointer to the RX_CONTEXT structure to use to get the device object and determine if this is a file system process.

### -param PostToFsp [out]

A pointer to a logical value that is set to TRUE on return if the request must be posted for later processing by the file system process.

## -returns

**RxStopMinirdr** returns STATUS_SUCCESS if the stop sequence was successful or one of the following error values:

| Return code | Description |
| ----------- | ----------- |
| STATUS_PENDING | The stop sequence for RDBSS and network mini-redirectors must be completed in the context of the file system process. If the call to **RxStopMinirdr** comes from a different process (a user-mode request, for example), then the request will be posted for later processing and STATUS_PENDING will be returned. This error can also be returned if certain internal RDBSS locks cannot be acquired without waiting. |
| STATUS_REDIRECTOR_HAS_OPEN_HANDLES | The network mini-redirector has open handles and cannot be stopped at this time. |
| STATUS_REDIRECTOR_STOPPED | The network mini-redirector was already stopped. |

## -remarks

When a stop request is issued to RDBSS, there may be ongoing requests in the RDBSS. Some of the requests can be canceled while the remaining requests need to be processed to completion.

There are a number of strategies that can be employed to close down the RDBSS. Currently, the most conservative approach is employed. The cancellation of those operations that can be canceled and the stop operation is held back until the remaining requests run through to completion.

The RDBSS **RxStopMinirdr** is usually called as a result of an FSCTL or IOCTL request from a user-mode application or service to stop the network mini-redirector, although this call could also be made from the network mini-redirector or as part of shutdown processing by the operating system.

Once a call to **RxStopMinirdr** is issued, the only operations allowed by RDBSS and passed to the network mini-redirector are requests for the following I/O request packets:

* IRP_MJ_CLEANUP
* IRP_MJ_CLOSE

The stop sequence for RDBSS and the network mini-redirector must be completed in the context of the file system process. If the call to **RxStopMinirdr** comes from a different process (a user-mode request, for example), then the request must be posted for later processing and STATUS_PENDING will be returned. In this case, the effective user ID (the logon ID) of the caller is saved in the **FsdUid** member of the **RxContext** parameter. In addition, If certain internal RDBSS locks cannot be obtained without waiting, STATUS_PENDING is returned and **PostToFsp** is set to TRUE. When STATUS_PENDING is returned, then **RxStopMinirdr** will be posted for later processing by a file system process and completed.

If a network mini-redirector indicates support for UNC when registering with RDBSS (the **Controls** parameter to **RxRegisterMinirdr**), then **RxStopMinirdr** will try to de-register the **DeviceName** of the network mini-redirector as a UNC provider with MUP (calls [**FsRtlDeregisterUncProvider**](../ntifs/nf-ntifs-fsrtlderegisteruncprovider.md) on behalf of the network mini-redirector). **RxStopMinirdr** also de-registers the file system with the I/O manager (calls [**IoUnregisterFileSystem**](../ntifs/nf-ntifs-iounregisterfilesystem.md)) on behalf of the network mini-redirector).

The **RxStopMinirdr** routine then calls the network mini-redirector **MrxStop** callback routine if this routine is implemented. If there are no active FCBs remaining, STATUS_SUCCESS is returned. If there are some remaining active FCBs, STATUS_REDIRECTOR_HAS_OPEN_HANDLES is returned. In either case, the RDBSS dispatcher for the redirector is spun down and the internal state of the network mini-redirector in RDBSS is set to RDBSS_STARTABLE.

## -see-also

[**FsRtlDeregisterUncProvider**](../ntifs/nf-ntifs-fsrtlderegisteruncprovider.md)

[**IoUnregisterFileSystem**](../ntifs/nf-ntifs-iounregisterfilesystem.md)

[**RxRegisterMinirdr**](nf-mrx-rxregisterminirdr.md)

[**RxStartMiniRdr**](nf-mrx-rxstartminirdr.md)

[**RxUnregisterMinirdr**](../rxstruc/nf-rxstruc-rxunregisterminirdr.md)

[**RxpUnregisterMinirdr**](nf-mrx-rxpunregisterminirdr.md)
