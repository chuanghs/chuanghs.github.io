
@Api(value="/dispatch_car", tags = "派車單資訊")
@Tag(name = "派車單資訊")

@ApiOperation(value = "已匯入 ERP 派車單", notes = "取得指定期間已匯入 ERP 之派車單")
@Operation(summary = "已匯入 ERP 派車單", description = "取得指定期間已匯入 ERP 之派車單")

@ApiParam(required = true, value = "中心代碼")
@Parameter(required = true, description = "中心代碼")

@ApiModel(description = "請購單明細")
@Schema(description = "請購單明細")

@ApiModelProperty(value="表單單號")
 @Schema(description="表單單號")


@ApiResponse(code=200, message = "", response = Result.class)
    @ApiResponse(responseCode="200", description = "", content = {
        @Content(mediaType = MediaType.APPLICATION_JSON, schema = @Schema(allOf = {Result.class, LeaveSummary.class})),
        @Content(mediaType = MediaType.APPLICATION_XML, schema = @Schema(allOf = {Result.class, LeaveSummary.class}))
    })

