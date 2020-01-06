# Sql Server to Azure Event Hub using Debezium

## Scope

* Sql Server 2019 running on windows server 2019 data center edition VM on Azure
* DDL for the table:
```
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[FareTypes](
	[FareTypeId] [int] NOT NULL,
	[FareTypeAbbr] [varchar](10) NULL,
	[Description] [varchar](30) NULL,
	[FlatFareAmt] [real] NULL,
	[FlatRate] [real] NULL,
	[DistanceRate] [real] NULL,
	[PolygonRate] [real] NULL,
	[TimeRate] [real] NULL,
	[PolyTypeId] [int] NULL,
	[PrepaidReq] [char](2) NULL,
	[FareAdjustDiffZon] [float] NULL,
	[FareAdjustSameZon] [float] NULL,
	[DistanceLimitSameZon] [float] NULL,
	[DistanceLimitDiffZon] [float] NULL,
	[NumCode] [smallint] NULL,
	[FareMode] [smallint] NULL,
	[FareCalcType] [smallint] NULL,
	[VariationOf] [int] NULL,
	[MinFare] [real] NULL,
	[MaxFare] [real] NULL,
	[RoundFare] [tinyint] NULL,
	[DistanceLimit] [int] NULL,
	[RoundDirection] [smallint] NULL,
	[Accuracy] [real] NULL,
	[InActive] [smallint] NULL,
	[Timestamp] [datetime2] DEFAULT 50SYSUTCDATETIME ( )
 CONSTRAINT [pkFareTypes] PRIMARY KEY CLUSTERED 
(
	[FareTypeId] ASC
) WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO
```
* Sample data for the table:
```
GO
INSERT [dbo].[FareTypes] ([FareTypeId], [FareTypeAbbr], [Description], [FlatFareAmt], [FlatRate], [DistanceRate], [PolygonRate], [TimeRate], [PolyTypeId], [PrepaidReq], [FareAdjustDiffZon], [FareAdjustSameZon], [DistanceLimitSameZon], [DistanceLimitDiffZon], [NumCode], [FareMode], [FareCalcType], [VariationOf], [MinFare], [MaxFare], [RoundFare], [DistanceLimit], [RoundDirection], [Accuracy], [InActive]) VALUES (1, N'STA', N'Standard Fare', NULL, 2, 0, 0, 0, 0, NULL, 0, 0, 0, 0, 1, 0, 0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL)
GO
INSERT [dbo].[FareTypes] ([FareTypeId], [FareTypeAbbr], [Description], [FlatFareAmt], [FlatRate], [DistanceRate], [PolygonRate], [TimeRate], [PolyTypeId], [PrepaidReq], [FareAdjustDiffZon], [FareAdjustSameZon], [DistanceLimitSameZon], [DistanceLimitDiffZon], [NumCode], [FareMode], [FareCalcType], [VariationOf], [MinFare], [MaxFare], [RoundFare], [DistanceLimit], [RoundDirection], [Accuracy], [InActive]) VALUES (2, N'CON', N'Concessionary Fare', NULL, 1, 0, 0, 0, 0, NULL, 0, 0, 0, 0, 2, 0, 0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL)
GO
INSERT [dbo].[FareTypes] ([FareTypeId], [FareTypeAbbr], [Description], [FlatFareAmt], [FlatRate], [DistanceRate], [PolygonRate], [TimeRate], [PolyTypeId], [PrepaidReq], [FareAdjustDiffZon], [FareAdjustSameZon], [DistanceLimitSameZon], [DistanceLimitDiffZon], [NumCode], [FareMode], [FareCalcType], [VariationOf], [MinFare], [MaxFare], [RoundFare], [DistanceLimit], [RoundDirection], [Accuracy], [InActive]) VALUES (3, N'CSF', N'Child Standard Fare', NULL, 1, 0, 0, 0, 0, NULL, 0, 0, 0, 0, 3, 0, 0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL)
GO
INSERT [dbo].[FareTypes] ([FareTypeId], [FareTypeAbbr], [Description], [FlatFareAmt], [FlatRate], [DistanceRate], [PolygonRate], [TimeRate], [PolyTypeId], [PrepaidReq], [FareAdjustDiffZon], [FareAdjustSameZon], [DistanceLimitSameZon], [DistanceLimitDiffZon], [NumCode], [FareMode], [FareCalcType], [VariationOf], [MinFare], [MaxFare], [RoundFare], [DistanceLimit], [RoundDirection], [Accuracy], [InActive]) VALUES (4, N'CCF', N'Child Concessionary Fare', NULL, 0.5, 0, 0, 0, 0, NULL, 0, 0, 0, 0, 4, 0, 0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL)
GO
INSERT [dbo].[FareTypes] ([FareTypeId], [FareTypeAbbr], [Description], [FlatFareAmt], [FlatRate], [DistanceRate], [PolygonRate], [TimeRate], [PolyTypeId], [PrepaidReq], [FareAdjustDiffZon], [FareAdjustSameZon], [DistanceLimitSameZon], [DistanceLimitDiffZon], [NumCode], [FareMode], [FareCalcType], [VariationOf], [MinFare], [MaxFare], [RoundFare], [DistanceLimit], [RoundDirection], [Accuracy], [InActive]) VALUES (5, N'ESC', N'Escort Fare', NULL, 0.75, 0, 0, 0, 0, NULL, 0, 0, 0, 0, 5, 0, 0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL)
GO
INSERT [dbo].[FareTypes] ([FareTypeId], [FareTypeAbbr], [Description], [FlatFareAmt], [FlatRate], [DistanceRate], [PolygonRate], [TimeRate], [PolyTypeId], [PrepaidReq], [FareAdjustDiffZon], [FareAdjustSameZon], [DistanceLimitSameZon], [DistanceLimitDiffZon], [NumCode], [FareMode], [FareCalcType], [VariationOf], [MinFare], [MaxFare], [RoundFare], [DistanceLimit], [RoundDirection], [Accuracy], [InActive]) VALUES (6, N'FOC', N'Free Of Charge', NULL, 0, 0, 0, 0, 0, NULL, 0, 0, 0, 0, 6, 0, 0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL)
GO
```
* Implement high-availability [debezium](https://debezium.io/) automated setup running on Azure AKS to publish message to Azure Event Hub
```
{
  "table": "FareTypes",
  "action": "insert|update|delete",
  "timestamp": "2002-10-02T15:00:00.05Z",
  "id": "single field pk field value",
  "data": {
	  "FareTypeId": 1,
	  "FareTypeAbbr": "STA",
	  "Description": "Standard Fare",
	  "FlatRate": 2.0,
	  "DistanceRate": 0.0,
	  "PolygonRate": 0.0,
	  "TimeRate": 0.0,
	  "PolyTypeId": 0,
	  "FareAdjustDiffZon": 0.0,
	  "FareAdjustSameZon": 0.0,
	  "DistanceLimitSameZon": 0.0,
	  "DistanceLimitDiffZon": 0.0,
	  "NumCode": 1,
	  "FareMode": 0,
	  "FareCalcType": 0,
	  "SourceTimestamp": "2002-10-02T15:00:00.05Z"
  }
}
```
* Create a stress test using JMeter to test performance with 1, 10, 25, 50 thread users
* Markdown documentation for instructions
