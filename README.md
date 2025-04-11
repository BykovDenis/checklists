import React, { useCallback, useEffect, useState } from 'react';
import { connect } from 'react-redux';
import { bindActionCreators, compose } from 'redux';

import FlexContainer from '@sber-risks-ui/core/flex-container';
import FormControl from '@sber-risks-ui/core/form-control';
import Typography from '@sber-risks-ui/core/typography';

import AccessDenied from '../../components/access-denied';
import FormContainer from '../../components/common/form-container';
import HeaderRowContainer from '../../components/common/header-row-container';
import Paper from '../../components/common/paper';
import CreditCurveMappingListDirectory from '../../components/directories/credit-curve-mapping-list-directory';
import DirectoriesSearchLineContainer from '../../components/directories/directories-search-line-container';
import DirectorySearchLine from '../../components/directories/directory-search-line';
import { CreditCurveMappingTitle, DirectoryTitle } from '../../components/formatted-messages';
import LoadProgress from '../../components/load-progress';
import MainContainer from '../../components/main-container';
import MainNavigation from '../../components/main-navigation';
import NoDataComponent from '../../components/no-data';
import sortObjectData from '../../helpers/sort-object-data';
import withInitialization from '../../hocs/with-initialization';
import { getAccessParams } from '../../redux/actions';
import {
  creditCurveMappingListDirectoryCalculatePagesCount,
  creditCurveMappingListDirectoryChangePage,
  creditCurveMappingListDirectoryChangeRowPerPage,
  creditCurveMappingListDirectoryTableSort,
  loadCreditCurveMappingData,
  loadCreditCurveMappingDictionary,
} from '../../redux/credit-curve-mapping/actions';
import {
  getCreditCurveMappingListDirectoryPaginator,
  getCreditCurveMappingListLoadStatus,
  getCreditCurveMappingListSortByField,
  getCreditCurveMappingListSortDirection,
  getCreditCurveMappingListSpreadSelector,
  getRouteName,
  getRowsPerPage,
} from '../../redux/credit-curve-mapping/selectors';
import LoadStatus from '../../enums/load-status';

const CreditCurveMappingDirectory = (props: any) => {
  const [creditCurveMapping, setCreditCurveMapping] = useState<any[]>([]);
  const [loadState, setLoadState] = useState<string>(LoadStatus.initial);
  const [searchText, setSearchText] = useState<string | null>(null);

  useEffect(() => {
    props.loadCreditCurveMappingDictionary();
    setCreditCurveMapping(props.creditCurveMappingListDirectory || []);
  }, []);

  const calculatePagesCount = useCallback(() => {
    const countPage = props.creditCurveMappingListDirectory?.length || 0;
    props.creditCurveMappingListDirectoryCalculatePagesCount(countPage);
  }, [props.creditCurveMappingListDirectory]);

  const pageChangeHandler = (evt: any, page: number) => {
    props.creditCurveMappingListDirectoryChangePage(page);
    calculatePagesCount();
  };

  const rowsPerPageChangeHandler = (evt: any) => {
    props.creditCurveMappingListDirectoryChangeRowPerPage(evt.target.value);
    calculatePagesCount();
  };

  const removeListItemHandler = () => {
    setCreditCurveMapping([]);
    setLoadState(LoadStatus.initial);
  };

  const onTableRowClick = (evt: any) => {
    const index = evt.currentTarget.dataset.index;
    if (index !== undefined) {
      const mapped = creditCurveMapping.map((el, idx) => {
        if (idx === parseInt(index, 10)) {
          el.topElement.isCollapsed = !el.topElement.isCollapsed;
        }
        return el;
      });
      setCreditCurveMapping(mapped);
      setLoadState(LoadStatus.loaded);
    }
    evt.stopPropagation();
  };

  const onCreditCurveMappingListDirectoryTableSort = (itemName: string) => {
    const sorted = sortObjectData(
      props.creditCurveMappingListDirectory,
      itemName,
      props.sortDirection
    );
    setCreditCurveMapping(sorted);
    setLoadState(LoadStatus.loaded);
    props.creditCurveMappingListDirectoryTableSort(itemName);
  };

  const _creditCurveMappingSearchByAllFields = (list: any[], searchVal: any) => {
    const search = searchVal.trim().toUpperCase();
    return list.filter((el: any) => {
      const { startDate, createdDate, country, currency, source, createdByUser } = el.topElement;
      return (
        startDate === search ||
        createdDate === search ||
        country?.toUpperCase().includes(search) ||
        currency?.toUpperCase().includes(search) ||
        source?.toUpperCase().includes(search) ||
        createdByUser?.toUpperCase().includes(search)
      );
    });
  };

  const onInputSearchCreditCurveMappingChange = (searchValue: any) => {
    const result = _creditCurveMappingSearchByAllFields(
      props.creditCurveMappingListDirectory,
      searchValue
    );
    setCreditCurveMapping(result);
    setLoadState(LoadStatus.loaded);
    setSearchText(searchValue);
  };

  const _creditCurveMappingDictionaryData = () => {
    const { page, rowsPerPage } = props.paginator;
    const list = loadState === LoadStatus.initial ? props.creditCurveMappingListDirectory : creditCurveMapping;
    if (list.length <= rowsPerPage * page && page > 0) {
      props.creditCurveMappingListDirectoryChangePage(0);
    }
    return (
      <FlexContainer flexDirection="column" alignItems="flex-start" margin="30px 0 0 0">
        <CreditCurveMappingListDirectory
          creditCurveMappingCount={list.length}
          sortDirection={props.sortDirection}
          sortByField={props.sortByField}
          paginator={props.paginator}
          creditCurveMappingItems={list}
          creditCurveMappingListDirectoryTableSort={onCreditCurveMappingListDirectoryTableSort}
          creditCurveMappingListDirectoryToggleItem={props.creditCurveMappingListDirectoryToggleItem}
          removeListItemHandler={removeListItemHandler}
          onTableRowClick={onTableRowClick}
          directories={props.directories}
          userAccess={props.userAccess}
          onPageChange={pageChangeHandler}
          onRowsPerPageChange={rowsPerPageChangeHandler}
          rowsPerPageOptions={props.rowsPerPage}
          rowsPerPage={rowsPerPage}
        />
        {list.length === 0 ? <NoDataComponent /> : null}
      </FlexContainer>
    );
  };

  const onTenantChangeApply = () => {
    props.getAccessParams(props.routeName, () => {
      props.loadCreditCurveMappingData();
      setCreditCurveMapping([]);
      setLoadState(LoadStatus.initial);
    }, true);
  };

  const {
    loadStatusCreditCurveMapping,
    userAccess,
    loadStatusAccessData
  } = props;
  const { status } = loadStatusCreditCurveMapping;
  const isLoading = status === LoadStatus.load;
  const isAccessDenied = userAccess.isAccessDenied;
  const isAccessDeniedComponent = userAccess.creditCurveMappingDirectoryRoute.action !== 'SHOW';
  const isLoadStatusAccessLoaded = loadStatusAccessData.status === LoadStatus.loaded;
  const isLoadStatusAccessLoad = loadStatusAccessData.status === LoadStatus.load;
  const isLoadStatusAccessInitial = loadStatusAccessData.status === LoadStatus.initial;
  const isLoadStatusAccessLoadedFailed = loadStatusAccessData.status === LoadStatus.loadingFailed;

  return (
    <>
      <MainNavigation onTenantChangeApply={onTenantChangeApply}>
        <HeaderRowContainer id="credit-curce-mapping-directory">
          <FormContainer position="relative" left={15} top={0}>
            <FormControl>
              <Typography variant="H1">
                <DirectoryTitle />: <CreditCurveMappingTitle />
              </Typography>
              {!isAccessDenied && !isAccessDeniedComponent && (
                <DirectoriesSearchLineContainer>
                  <DirectorySearchLine
                    id="creditCurveMapping-directory-search"
                    name="creditCurveMapping-directory-search"
                    searchString={searchText}
                    onSearchStringChange={onInputSearchCreditCurveMappingChange}
                  />
                </DirectoriesSearchLineContainer>
              )}
            </FormControl>
          </FormContainer>
        </HeaderRowContainer>
        <MainContainer loadStatusUserData={props.loadStatusUserData}>
          <Paper>
            {(isLoading || isLoadStatusAccessLoad) && (
              <FlexContainer margin="10px 0 0 0">
                <LoadProgress />
              </FlexContainer>
            )}
            {!isLoading && !isAccessDenied && !isAccessDeniedComponent && _creditCurveMappingDictionaryData()}
            {(isLoadStatusAccessLoaded || isLoadStatusAccessLoadedFailed) && isAccessDenied && (
              <AccessDenied message="Система Loan Pricing" comments={loadStatusAccessData.message} />
            )}
            {!isLoadStatusAccessInitial && isLoadStatusAccessLoaded && isAccessDeniedComponent && (
              <AccessDenied
                message="Credit curve mapping directory"
                comments={userAccess.creditCurveMappingDirectoryRoute.message}
              />
            )}
          </Paper>
        </MainContainer>
      </MainNavigation>
    </>
  );
};

const mapStateToProps = (state: any) => ({
  creditCurveMappingListDirectory: getCreditCurveMappingListSpreadSelector(state),
  paginator: getCreditCurveMappingListDirectoryPaginator(state),
  sortDirection: getCreditCurveMappingListSortDirection(state),
  sortByField: getCreditCurveMappingListSortByField(state),
  rowsPerPage: getRowsPerPage(state),
  tradeIds: state.tradeIds,
  directories: state.directories,
  loadStatusCreditCurveMapping: getCreditCurveMappingListLoadStatus(state),
  routeName: getRouteName(state),
});

const mapDispatchToProps = (dispatch: any) => bindActionCreators({
  creditCurveMappingListDirectoryChangeRowPerPage,
  creditCurveMappingListDirectoryChangePage,
  creditCurveMappingListDirectoryCalculatePagesCount,
  creditCurveMappingListDirectoryTableSort,
  loadCreditCurveMappingDictionary,
  loadCreditCurveMappingData,
  getAccessParams,
}, dispatch);

export default compose(connect(mapStateToProps, mapDispatchToProps), withInitialization)(CreditCurveMappingDirectory);
