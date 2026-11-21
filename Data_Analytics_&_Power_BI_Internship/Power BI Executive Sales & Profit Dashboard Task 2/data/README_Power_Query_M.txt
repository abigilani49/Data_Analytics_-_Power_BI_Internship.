// OPTIONAL: Load the ORIGINAL file directly instead of using the cleaned CSVs.
// Orders only (first 9,994 rows), then derive dimensions via Reference queries.
let
  Source   = Csv.Document(File.Contents("C:\path\superstore_sales__1__power_bi.csv"),[Delimiter=",",Encoding=1252,QuoteStyle=QuoteStyle.Csv]),
  Promoted = Table.PromoteHeaders(Source,[PromoteAllScalars=true]),
  Orders   = Table.FirstN(Promoted, 9994),
  Typed    = Table.TransformColumnTypes(Orders,{{"Order Date",type date},{"Ship Date",type date},{"Sales",type number},{"Quantity",Int64.Type},{"Discount",type number},{"Profit",type number}},"en-US")
in Typed
