# Logging

Data streams from Harp devices can be easily logged in real time. Since all communication between a device and the host is made via a sequence of [`HarpMessage`](xref:Bonsai.Harp.HarpMessage) objects, this is the only piece of information we need to reconstruct the state of the experiment, as seen by the Harp device, at any given point in time.

Moreover, since all Harp messages follow a simple [binary protocol](../protocol/BinaryProtocol-8bit.md), they can be efficiently logged into disk using a flat binary file. The next sections will cover how to do this and discuss current recommendations for logging data streams from Harp devices.

## Log to a single binary file

Any `HarpMessage` can be logged by simply saving its raw binary representation. This raw binary representation can be accessed as a `byte[]` via the [`MessageBytes`](xref:Bonsai.Harp.HarpMessage.MessageBytes) property. This means we can record the entire raw binary stream by simply passing the sequence of message bytes to a binary logger.

The `Bonsai.Harp` package provides a dedicated [`MessageWriter`](xref:Bonsai.Harp.MessageWriter) operator that easily encapsulates this functionality:

:::workflow
![LogAllMessages](~/workflows/log-all-messages.bonsai)
:::

Since all logging takes place on top of arbitrary `HarpMessage` streams, the `MessageWriter` operator can be used to log multiple devices in parallel, log message streams filtered using the [`FilterRegister`](xref:Bonsai.Harp.FilterRegister) operator, or even save host-generated commands such as messages generated by the [`CreateMessage`](xref:Bonsai.Harp.CreateMessage) operator.

> [!Warning]
> While logging all Harp messages to a single flat binary file is very easy, it is not always the most convenient way to log data for post-processing and analysis, as it requires examining the header of each and every message to determine how to process its contents.

## Log to separate files per register

We can use the [`GroupByRegister`](xref:Bonsai.Harp.GroupByRegister) operator to automatically split the single Harp message stream into independent sub-streams for each device register address. When `MessageWriter` receives such a split message sequence, it will automatically generate one independent file for each register.

Since the payload stored in any single register always has the same format and size, this is enough to ensure we can read and parse the entire single-register binary file in one bulk operation. A simple implementation of this pattern is shown below:

:::workflow
![LogDemux](~/workflows/log-demux.bonsai)
:::

These single-register log files can then be loaded using the [Data Interface](python.md). The `device.yml` metadata file for each device is always available in the root folder of its source repository.

> [!Tip]
> The `device.yml` metadata file for each device can also be recovered at runtime using the `GetMetadata` operator included with most device packages and saved using the [`WriteAllText`](xref:Bonsai.IO.WriteAllText) operator.