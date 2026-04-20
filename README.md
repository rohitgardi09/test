
public List<String[]> parseFile(InputStream inputStream, String fileKey) {
    logger.info("Starting Parsing Excel File: {}", fileKey);

    DataFormatter dataFormatter = new DataFormatter();
    List<String[]> rows = new LinkedList<>();

    try (Workbook workbook = WorkbookFactory.create(inputStream)) {

        Sheet sheet = workbook.getSheetAt(0);

        // Use header to determine column count
        int columnCount = 0;
        Row headerRow = sheet.getRow(0);
        if (headerRow != null) {
            columnCount = headerRow.getLastCellNum();
        }

        for (Row row : sheet) {

            if (row == null) continue;

            List<String> cellValues = new LinkedList<>();

            for (int i = 0; i < columnCount; i++) {

                Cell cell = row.getCell(i, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);

                String value;

                // ✅ Only numeric → BigDecimal → plain string
                if (cell.getCellType() == CellType.NUMERIC && !DateUtil.isCellDateFormatted(cell)) {
                    value = new BigDecimal(cell.toString()).toPlainString();
                } else {
                    // ✅ Everything else (string, blank, boolean, formula, date)
                    value = dataFormatter.formatCellValue(cell);
                }

                cellValues.add(value);
            }

            rows.add(cellValues.toArray(new String[0]));
        }

        logger.info("Parsing Excel file completed: {}", fileKey);
        return rows;

    } catch (Exception e) {
        logger.error("Parsing Excel file failed for fileKey: {}", fileKey, e);
    }

    return Collections.emptyList();
}




***********

public List<String[]> parseFile(InputStream inputStream, String fileKey) {
    logger.info("Starting Parsing Excel File: {}", fileKey);

    DataFormatter dataFormatter = new DataFormatter();
    List<String[]> rows = new LinkedList<>();

    try (Workbook workbook = WorkbookFactory.create(inputStream)) {

        Sheet sheet = workbook.getSheetAt(0);

        // Get column count from header (recommended)
        int columnCount = 0;
        Row headerRow = sheet.getRow(0);
        if (headerRow != null) {
            columnCount = headerRow.getLastCellNum();
        }

        for (Row row : sheet) {

            // Skip completely null rows (safety)
            if (row == null) {
                continue;
            }

            List<String> cellValues = new LinkedList<>();

            logger.debug("Processing row number: {}", row.getRowNum());

            for (int i = 0; i < columnCount; i++) {

                // Handle missing/empty cells
                Cell cell = row.getCell(i, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);

                String value = dataFormatter.formatCellValue(cell);
                cellValues.add(value);
            }

            rows.add(cellValues.toArray(new String[0]));
        }

        logger.info("Parsing Excel file completed: {}", fileKey);
        return rows;

    } catch (Exception e) {
        logger.error("Parsing Excel file failed for fileKey: {}", fileKey, e);
    }

    return Collections.emptyList();
}




Charges Configured in Report and Merchant for Release Testing – QA Can Reach Out for Demo or Any Further Explanation
