## EnslavedTable Component
The `EnslavedTable` component is a React functional component that represents a table for displaying data related to the Enslaved dataset. It includes various dependencies, Redux state management, and event handlers.

## Dependencies
The component imports the following dependencies:

- `React`: The core React library.
- `ag-grid-react`: Provides the `AgGridReact` component for rendering the data table.
- `react-redux`: Provides the `useDispatch` and `useSelector` hooks for Redux state management.
- `@/redux/store`: Imports the Redux store and provides the AppDispatch and RootState types.
- `../../FunctionComponents/CustomHeader`: Imports the `CustomHeader` component for customizing table headers.
- `@/utils/functions/generateRowsData`: Imports a utility function for generating row data based on the Enslaved dataset.
- `@/redux/getTableSlice`: Imports Redux actions for setting the column definitions, row data, and data in the table.
- `@/redux/getColumnSlice`: Imports Redux actions for setting the visible columns in the table.
- `@/utils/functions/getBreakPoints`: Imports a utility function for getting the number of rows per page based on the window size.
- `@react-hook/window-size`: Provides a hook for getting the window size.
- `@mui/material`: Provides components such as Pagination, Skeleton, and TablePagination from the Material-UI library.
- `ag-grid-community`: Provides interfaces and styles for the ag-Grid library.
- `@/fetchAPI/pastEnslavedApi/fetchEnslavedOptionsList`: Imports a function for fetching options related to the Enslaved dataset.
- `@/utils/functions/hasValueGetter`: Imports a utility function for checking if a column has a value getter.

# Usage
To use the `EnslavedTable` component, follow these steps:

Import the necessary dependencies and components:

```jsx
import React, {
  CSSProperties,
  useCallback,
  useEffect,
  useMemo,
  useRef,
  useState,
} from 'react';
import { AgGridReact } from 'ag-grid-react';
import { useDispatch, useSelector } from 'react-redux';
import { AppDispatch, RootState } from '@/redux/store';
import CustomHeader from '../../FunctionComponents/CustomHeader';
import { generateRowsData } from '@/utils/functions/generateRowsData';
import { setColumnDefs, setRowData, setData } from '@/redux/getTableSlice';
import { setVisibleColumn } from '@/redux/getColumnSlice';
import { getRowsPerPage } from '@/utils/functions/getBreakPoints';
import { useWindowSize } from '@react-hook/window-size';
import { Pagination, Skeleton, TablePagination } from '@mui/material';
import {
  ColumnDef,
  StateRowData,
  TableCellStructureInitialStateProp,
  VoyageOptionsGropProps,
  VoyageTableCellStructure,
} from '@/share/InterfaceTypesTable';
import {
  AutoCompleteInitialState,
  CurrentPageInitialState,
  RangeSliderState,
  TYPESOFDATASETPEOPLE,
} from '@/share/InterfaceTypes';
import ENSLAVED_TABLE from '@/utils/flatfiles/enslaved_table_cell_structure.json';
import AFRICANORIGINS_TABLE from '@/utils/flatfiles/african_origins_table_cell_structure.json';
import TEXAS_TABLE from '@/utils/flatfiles/texas_table_cell_structure.json';
import { ICellRendererParams } from 'ag-grid-community';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-alpine.css';
import '@/style/table.scss';
import { fetchEnslavedOptionsList } from '@/fetchAPI/pastEnslavedApi/fetchEnslavedOptionsList';
import ButtonDropdownSelectorEnslaved from './ColumnSelectorEnslavedTable/ButtonDropdownSelectorEnslaved';
import { hasValueGetter } from '@/utils/functions/hasValueGetter';
```
- Implement the `EnslavedTable` component:
```jsx
const EnslavedTable: React.FC = () => {
  // Redux state and dispatch
  const dispatch: AppDispatch = useDispatch();
  const { columnDefs, data, rowData } = useSelector(
    (state: RootState) => state.getTableData as StateRowData
  );
  const { rangeSliderMinMax: rang, varName } = useSelector(
    (state: RootState) => state.rangeSlider as RangeSliderState
  );
  const { autoCompleteValue, autoLabelName } = useSelector(
    (state: RootState) => state.autoCompleteList as AutoCompleteInitialState
  );
  const { currentPage } = useSelector(
    (state: RootState) => state.getScrollPage as CurrentPageInitialState
  );
  const { visibleColumnCells } = useSelector(
    (state: RootState) => state.getColumns as TableCellStructureInitialStateProp
  );
  const {
    dataSetKeyPeople,
    dataSetValuePeople,
    dataSetValueBaseFilter,
    styleNamePeople,
    tableFlatfile: tableFileName,
  } = useSelector((state: RootState) => state.getPeopleDataSetCollection);

  const [loading, setLoading] = useState(false);
  const [page, setPage] = useState<number>(0);
  const [rowsPerPage, setRowsPerPage] = useState(
    getRowsPerPage(window.innerWidth, window.innerHeight)
  );
  const [totalResultsCount, setTotalResultsCount] = useState(0);
  const gridRef = useRef<any>(null);
  const [tablesCell, setTableCell] = useState<VoyageTableCellStructure[]>([]);
  const { currentEnslavedPage } = useSelector(
    (state: RootState) => state.getScrollEnslavedPage
  );
  const [width, height] = useWindowSize();
  const maxWidth =
    width > 1024
      ? width > 1440
        ? width * 0.88
        : width * 0.92
      : width === 1024
      ? width * 0.895
      : width < 768
      ? width * 0.8
      : width * 0.75;
  const [style, setStyle] = useState({
    width: maxWidth,
    height: height * 0.62,
  });

  const containerStyle = useMemo(
    () => ({ width: maxWidth, height: height * 0.7 }),
    [maxWidth, height]
  );

  useEffect(() => {
    const loadTableCellStructure = async () => {
      try {
        // ** TODO Need to refactor later
        if (styleNamePeople === TYPESOFDATASETPEOPLE.allEnslaved) {
          setTableCell(ENSLAVED_TABLE.cell_structure);
        } else if (styleNamePeople === TYPESOFDATASETPEOPLE.africanOrigins) {
          setTableCell(AFRICANORIGINS_TABLE.cell_structure);
        } else if (styleNamePeople === TYPESOFDATASETPEOPLE.texas) {
          setTableCell(TEXAS_TABLE.cell_structure);
        }
      } catch (error) {
        console.error('Failed to load table cell structure:', error);
      }
    };
    loadTableCellStructure();
  }, [tableFileName, styleNamePeople]);

  useEffect(() => {
    if (tablesCell.length > 0) {
      const visibleColumns = tablesCell
        .filter((cell: any) => cell.visible)
        .map((cell: any) => cell.colID);
      dispatch(setVisibleColumn(visibleColumns));
    }
  }, [tablesCell, dispatch]);

  useEffect(() => {
    const handleResize = () => {
      setRowsPerPage(getRowsPerPage(window.innerWidth, window.innerHeight));
    };
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  useEffect(() => {
    setStyle({
      width: maxWidth,
      height: height * 0.65,
    });
  }, [width, height, maxWidth]);

  const saveDataToLocalStorage = useCallback(
    (data: VoyageOptionsGropProps[], visibleColumnCells: string[]) => {
      localStorage.setItem('data', JSON.stringify(data));
      localStorage.setItem(
        'visibleColumnCells',
        JSON.stringify(visibleColumnCells)
      );
    },
    []
  );

  useEffect(() => {
    saveDataToLocalStorage(data, visibleColumnCells);
  }, [data]);

  useEffect(() => {
    let subscribed = true;
    const fetchData = async () => {
      setLoading(true);
      const newFormData: FormData = new FormData();
      newFormData.append('results_page', String(page + 1));
      newFormData.append('results_per_page', String(rowsPerPage));

      if (rang[varName] && currentEnslavedPage === 2) {
        newFormData.append(varName, String(rang[varName][0]));
        newFormData.append(varName, String(rang[varName][1]));
      }

      if (autoCompleteValue && varName) {
        for (let i = 0; i < autoLabelName.length; i++) {
          const label = autoLabelName[i];
          newFormData.append(varName, label);
        }
      }

      if (styleNamePeople !== TYPESOFDATASETPEOPLE.allEnslaved) {
        for (const value of dataSetValuePeople) {
          newFormData.append(dataSetKeyPeople, String(value));
        }
      }

      try {
        const response = await dispatch(
          fetchEnslavedOptionsList(newFormData)
        ).unwrap();
        if (subscribed) {
          setTotalResultsCount(Number(response.headers.total_results_count));

          dispatch(setData(response.data));
          saveDataToLocalStorage(response.data, visibleColumnCells);
        }
      } catch (error) {
        console.log('error', error);
        setLoading(false);
      } finally {
        setLoading(false);
      }
    };
    fetchData();
    return () => {
      subscribed = false;
    };
  }, [
    dispatch,
    rowsPerPage,
    page,
    currentPage,
    varName,
    rang,
    autoCompleteValue,
    autoLabelName,
    dataSetValuePeople,
    dataSetKeyPeople,
    dataSetValueBaseFilter,
    styleNamePeople,
    saveDataToLocalStorage,
    visibleColumnCells,
  ]);

  useEffect(() => {
    if (data.length > 0) {
      const finalRowData = generateRowsData(data, tableFileName);
      const newColumnDefs: ColumnDef[] = tablesCell.map(
        (value: VoyageTableCellStructure) => {
          const columnDef = {
            headerName: value.header_label,
            field: value.colID,
            sortable: true,
            autoHeight: true,
            wrapText: true,
            sortingOrder: value.order_by,
            headerTooltip: value.header_label,
            tooltipField: value.colID,
            hide: !visibleColumnCells.includes(value.colID),
            filter: true,
            cellRenderer: (params: ICellRendererParams) => {
              const values = params.value;
              if (Array.isArray(values)) {
                const style: CSSProperties = {
                  backgroundColor: '#e5e5e5',
                  borderRadius: '8px',
                  padding: '0px 10px',
                  height: '25px',
                  whiteSpace: 'nowrap',
                  overflow: 'hidden',
                  margin: '5px 0',
                  textAlign: 'center',
                  lineHeight: '25px',
                  fontSize: '13px',
                };
                const renderedValues = values.map(
                  (value: string, index: number) => (
                    <span key={`${index}-${value}`}>
                      <div style={style}>{`${value}\n`}</div>
                    </span>
                  )
                );
                return <div>{renderedValues}</div>;
              } else {
                return (
                  <div className="div-value">
                    <div className="value">{values}</div>
                  </div>
                );
              }
            },
            valueGetter: (params: ICellRendererParams) =>
              hasValueGetter(params, value),
          };
          return columnDef;
        }
      );

      dispatch(setColumnDefs(newColumnDefs));
      dispatch(setRowData(finalRowData as Record<string, any>[]));
    }
  }, [data, visibleColumnCells, dispatch]);

  const defaultColDef = useMemo(
    () => ({
      sortable: true,
      resizable: true,
      filter: true,
      initialWidth: 200,
      wrapHeaderText: true,
      autoHeaderHeight: true,
    }),
    []
  );

  const components = useMemo(
    () => ({
      agColumnHeader: CustomHeader,
    }),
    []
  );

  const getRowRowStyle = useCallback(
    () => ({
      fontSize: 13,
      fontWeight: 500,
      color: '#000',
      fontFamily: 'Roboto',
      paddingLeft: '20px',
    }),
    []
  );

  const handleColumnVisibleChange = useCallback(
    (params: any) => {
      const { columnApi } = params;
      const allColumns = columnApi.getAllColumns();
      const visibleColumns = allColumns
        .filter((column: any) => column.isVisible())
        .map((column: any) => column.getColId());

      dispatch(setVisibleColumn(visibleColumns));
    },
    [dispatch]
  );

  const gridOptions = useMemo(
    () => ({
      headerHeight: 40,
      suppressHorizontalScroll: true,
      onGridReady: (params: any) => {
        const { columnApi } = params;
        columnApi.autoSizeColumns();
      },
    }),
    []
  );

  const handleChangePage = useCallback((event: any, newPage: number) => {
    setPage(newPage);
  }, []);

  const handleChangeRowsPerPage = useCallback(
    (event: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
      setRowsPerPage(parseInt(event.target.value));
      setPage(0);
    },
    []
  );

  const handleChangePagePagination = useCallback(
    (event: any, newPage: number) => {
      setPage(newPage - 1);
    },
    []
  );

  return (
    <div>
      {loading ? (
        <div className="Skeleton-loading">
          <Skeleton />
          <Skeleton animation="wave" />
          <Skeleton animation={false} />
        </div>
      ) : (
        <div style={containerStyle} className="ag-theme-alpine grid-container">
          <div style={style}>
            <span className="tableContainer">
              <ButtonDropdownSelectorEnslaved />
              <TablePagination
                component="div"
                count={totalResultsCount}
                page={page}
                onPageChange={handleChangePage}
                rowsPerPageOptions={[5, 10, 12, 15, 20, 25, 30, 45, 50, 100]}
                rowsPerPage={rowsPerPage}
                onRowsPerPageChange={handleChangeRowsPerPage}
              />
            </span>

            <AgGridReact
              ref={gridRef}
              rowData={rowData}
              onColumnVisible={handleColumnVisibleChange}
              gridOptions={gridOptions}
              columnDefs={columnDefs}
              suppressMenuHide={true}
              animateRows={true}
              paginationPageSize={rowsPerPage}
              defaultColDef={defaultColDef}
              components={components}
              getRowStyle={getRowRowStyle}
              enableBrowserTooltips={true}
              tooltipShowDelay={0}
              tooltipHideDelay={1000}
            />
            <div className="pagination-div">
              <Pagination
                color="primary"
                count={Math.ceil(totalResultsCount / rowsPerPage)}
                page={page + 1}
                onChange={handleChangePagePagination}
              />
            </div>
          </div>
        </div>
      )}
    </div>
  );
};

export default EnslavedTable;

```
Export the `EnslavedTable` component as the default export of the file:

Use the `EnslavedTable` component in other parts of your application as needed.