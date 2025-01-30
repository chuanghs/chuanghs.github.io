---
layout: post
title: 轉換 swagger annotation 到 open api anootation
categories: java swagger openapi annotation
---

最近利用機會把 annotation 從 swagger 轉換到 openapi 的標準 annotation


| Swagger annotation | OpenAPI annotation |
| ------------------|--------------------|
| @Api(value="/url", tags = "標籤") | @Tag(name = "標籤") |
| @ApiOperation(value = "簡要說明", notes = "詳細說明") | @Operation(summary = "簡要說明", description = "詳細說明") |
| @ApiParam(required = true, value = "參數說明") |@Parameter(required = true, description = "參數說明") |
| @ApiModel(description = "data model 說明") | @Schema(description = "data model 說明") |
| @ApiModelProperty(value="data model 屬性說明") | @Schema(description="data model 屬性說明") |
| @ApiResponse(code=200, message = "", response = Result.class) | @ApiResponse(responseCode="200", description = "", content = { <br>    @Content(mediaType = MediaType.APPLICATION_JSON, schema = @Schema(allOf = {Result.class, LeaveSummary.class})), <br>     Content(mediaType = MediaType.APPLICATION_XML, schema = @Schema(allOf = {Result.class, LeaveSummary.class})) <br> }) |

