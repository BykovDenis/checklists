
import FlexContainer from '@sber-risks-ui/core/flex-container';
import FormControl from '@sber-risks-ui/core/form-control';
import Typography from '@sber-risks-ui/core/typography';
import React, {Component, Fragment} from 'react';
import {connect} from 'react-redux';
import {bindActionCreators, compose} from 'redux';

import AccessDenied from '../../components/access-denied';
import FormContainer from '../../components/common/form-container';
import HeaderRowContainer from '../../components/common/header-row-container';
import Paper from '../../components/common/paper';
import CreditCurveMappingListDirectory from '../../components/directories/credit-curve-mapping-list-directory';
import DirectoriesSearchLineContainer from '../../components/directories/directories-search-line-container';
import DirectorySearchLine from '../../components/directories/directory-search-line';
import {CreditCurveMappingTitle, DirectoryTitle} from '../../components/formatted-messages';
import LoadProgress from '../../components/load-progress';
import MainContainer from '../../components/main-container';
import MainNavigation from '../../components/main-navigation';
import NoDataComponent from '../../components/no-data';
import sortObjectData from '../../helpers/sort-object-data';
import withInitialization from '../../hocs/with-initialization';
import {getAccessParams} from '../../redux/actions';
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

class CreditCurveMappingDirectory extends Component<any, any> {
  constructor(props: any) {
    super(props);
    this.pageChangeHandler = this.pageChangeHandler.bind(this);
    this.rowsPerPageChangeHandler = this.rowsPerPageChangeHandler.bind(this);
    this.calculatePagesCount = this.calculatePagesCount.bind(this);
    this.onInputSearchCreditCurveMappingChange = this.onInputSearchCreditCurveMappingChange.bind(this);
    this.removeListItemHandler = this.removeListItemHandler.bind(this);
    this.onTenantChangeApply = this.onTenantChangeApply.bind(this);
    this.onCreditCurveMappingListDirectoryTableSort = this.onCreditCurveMappingListDirectoryTableSort.bind(this);
    this.onTableRowClick = this.onTableRowClick.bind(this);
    this.state = {creditCurveMapping: [], loadState: LoadStatus.initial, searchText: null};
  }

  componentDidMount() {
    const props: any = this.props;
    props.loadCreditCurveMappingDictionary();
    const creditCurveMappingData = props.creditCurveMappingListDirectory || [];
    this.setState({creditCurveMapping: creditCurveMappingData, loadState: LoadStatus.initial});
  }

  pageChangeHandler(evt: any, page: number) {
    const props: any = this.props;
    props.creditCurveMappingListDirectoryChangePage(page);
    this.calculatePagesCount();
  }

  rowsPerPageChangeHandler(evt: any) {
    const props: any = this.props;
    props.creditCurveMappingListDirectoryChangeRowPerPage(evt.target.value);
    this.calculatePagesCount();
  }

  calculatePagesCount() {
    const props: any = this.props;
    const countPage = (props.creditCurveMappingListDirectory && props.creditCurveMappingListDirectory.length) || 0;
    props.creditCurveMappingListDirectoryCalculatePagesCount(countPage);
  }

  removeListItemHandler() {
    this.setState({creditCurveMapping: [], loadState: LoadStatus.initial});
  }

  onTableRowClick(evt: any) {
    const props: any = this.props;
    const state: any = this.state;
    const element = evt.currentTarget;
    const {index} = element.dataset;
    const creditCurveMapping =
      (state.creditCurveMapping.length > 0 && state.creditCurveMapping) ||
      props.creditCurveMappingListDirectory;
    if (index) {
      const creditCurveMappingMapped = creditCurveMapping.map((element: any, elementIndex: number) => {
        if (elementIndex === parseInt(index, 10)) {
          element.topElement.isCollapsed = !element.topElement.isCollapsed;
        }
        return element;
      });
      this.setState({creditCurveMapping: creditCurveMappingMapped, loadState: LoadStatus.loaded});
    }
    evt.stopPropagation();
  }

  onCreditCurveMappingListDirectoryTableSort(itemName: string) {
    const props: any = this.props;
    const state: any = this.state;
    this.setState((state: any) => {
      const creditCurveMapping: any[] = state?.creditCurveMapping?.length > 0 ? props.creditCurveMappingListDirectory : [];
      const creditCurveMappingSorted = sortObjectData(creditCurveMapping, itemName, props.sortDirection);
      return {
        creditCurveMapping: creditCurveMappingSorted,
        loadState: LoadStatus.loaded,
      };
    });
    props.creditCurveMappingListDirectoryTableSort(itemName);
  }

  onInputSearchCreditCurveMappingChange(searchValue: unknown) {
    const props: any = this.props;
    const creditCurveMappingState = props.creditCurveMappingListDirectory || [];
    this.setState({
      creditCurveMapping: this._creditCurveMappingSearchByAllFields(creditCurveMappingState, searchValue),
      loadState: LoadStatus.loaded,
      searchText: searchValue,
    });
  }

  _creditCurveMappingSearchByAllFields(creditCurveMappingList: any, creditCurveMapping: any) {
    const searchParam: string = creditCurveMapping.trim().toUpperCase();
    return creditCurveMappingList.filter((creditCurveMappingElement: any) => {
      const startDate = creditCurveMappingElement.topElement.startDate;
      const createdDate = creditCurveMappingElement.topElement.createdDate;
      const country =
        creditCurveMappingElement.topElement.country && creditCurveMappingElement.topElement.country.toUpperCase();
      const currency =
        creditCurveMappingElement.topElement.currency && creditCurveMappingElement.topElement.currency.toUpperCase();
      const source =
        creditCurveMappingElement.topElement.source && creditCurveMappingElement.topElement.source.toUpperCase();
      const createByUser =
        creditCurveMappingElement.topElement.createdByUser &&
        creditCurveMappingElement.topElement.createdByUser.toUpperCase();
      return (
        startDate === searchParam ||
        createdDate === searchParam ||
        (country && country.indexOf(searchParam) > -1) ||
        (currency && currency.indexOf(searchParam) > -1) ||
        (source && currency.indexOf(searchParam) > -1) ||
        (createByUser && createByUser.indexOf(searchParam) > -1)
      );
    });
  }

  _creditCurveMappingDictionaryData() {
    const props: any = this.props;
    const state: any = this.state;
    const {page, rowsPerPage} = props.paginator;
    const creditCurveMapping =
      state.loadState === LoadStatus.initial
        ? props.creditCurveMappingListDirectory
        : state.creditCurveMapping;
    const creditCurveMappingCount: number = creditCurveMapping.length;

    if (creditCurveMapping && creditCurveMappingCount <= rowsPerPage * page && page > 0) {
      props.creditCurveMappingListDirectoryChangePage(0);
    }

    return (
      <FlexContainer flexDirection="column" alignItems="flex-start" margin="30px 0 0 0">
        <CreditCurveMappingListDirectory
          creditCurveMappingCount={creditCurveMappingCount}
          sortDirection={props.sortDirection as any}
          sortByField={props.sortByField}
          paginator={props.paginator}
          creditCurveMappingItems={creditCurveMapping}
          creditCurveMappingListDirectoryTableSort={this.onCreditCurveMappingListDirectoryTableSort}
          creditCurveMappingListDirectoryToggleItem={props.creditCurveMappingListDirectoryToggleItem}
          removeListItemHandler={this.removeListItemHandler}
          onTableRowClick={this.onTableRowClick}
          directories={props.directories}
          userAccess={props.userAccess}
          onPageChange={this.pageChangeHandler}
          onRowsPerPageChange={this.rowsPerPageChangeHandler}
          rowsPerPageOptions={props.rowsPerPage}
          rowsPerPage={rowsPerPage}
        />
        {creditCurveMapping.length === 0 ? <NoDataComponent/> : null}
      </FlexContainer>
    );
  }

  onTenantChangeApply() {
    const props: any = this.props;
    const cb = () => {
      props.loadCreditCurveMappingData();
      this.setState({creditCurveMapping: [], loadState: LoadStatus.initial});
    };
    props.getAccessParams(props.routeName, cb, true);
  }

  render() {
    const props: any = this.props;
    const state: any = this.state;
    const {loadStatusCreditCurveMapping, userAccess, loadStatusAccessData} = props;
    const {isAccessDenied, creditCurveMappingDirectoryRoute} = userAccess;
    const {status} = loadStatusCreditCurveMapping;
    const isLoading = status === LoadStatus.load;
    const {status: loadStatusAccess, message: loadStatusMessage} = loadStatusAccessData;
    const isAccessDeniedComponent = creditCurveMappingDirectoryRoute.action !== 'SHOW';
    const isLoadStatusAccessLoaded = loadStatusAccess === LoadStatus.loaded;
    const isLoadStatusAccessLoad = loadStatusAccess === LoadStatus.load;
    const isLoadStatusAccessInitial = loadStatusAccess === LoadStatus.initial;
    const isLoadStatusAccessLoadedFailed = loadStatusAccess === LoadStatus.loadingFailed;
    return (
      <Fragment>
        <MainNavigation onTenantChangeApply={this.onTenantChangeApply}>
          <HeaderRowContainer id="credit-curce-mapping-directory">
            <FormContainer position="relative" left={15} top={0}>
              <FormControl>
                <Typography variant="H1">
                  <DirectoryTitle/>: <CreditCurveMappingTitle/>
                </Typography>
                {!isAccessDenied && !isAccessDeniedComponent && (
                  <DirectoriesSearchLineContainer>
                    <DirectorySearchLine
                      id="creditCurveMapping-directory-search"
                      name="creditCurveMapping-directory-search"
                      searchString={state.searchText}
                      onSearchStringChange={this.onInputSearchCreditCurveMappingChange}
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
                  <LoadProgress/>
                </FlexContainer>
              )}
              {!isLoading && !isAccessDenied && !isAccessDeniedComponent && this._creditCurveMappingDictionaryData()}
              {(isLoadStatusAccessLoaded || isLoadStatusAccessLoadedFailed) && isAccessDenied && (
                <AccessDenied message="Система Loan Pricing" comments={loadStatusMessage}/>
              )}
              {!isLoadStatusAccessInitial && isLoadStatusAccessLoaded && isAccessDeniedComponent && (
                <AccessDenied
                  message="Credit curve mapping directory"
                  comments={creditCurveMappingDirectoryRoute.message}
                />
              )}
            </Paper>
          </MainContainer>
        </MainNavigation>
      </Fragment>
    );
  }
}

function mapStateToProps(state: any) {
  return {
    creditCurveMappingListDirectory: getCreditCurveMappingListSpreadSelector(state),
    paginator: getCreditCurveMappingListDirectoryPaginator(state),
    sortDirection: getCreditCurveMappingListSortDirection(state),
    sortByField: getCreditCurveMappingListSortByField(state),
    rowsPerPage: getRowsPerPage(state),
    tradeIds: state.tradeIds,
    directories: state.directories,
    loadStatusCreditCurveMapping: getCreditCurveMappingListLoadStatus(state),
    routeName: getRouteName(state),
  };
}

function mapDispatchToProps(dispatch: any) {
  const bindObject = {
    creditCurveMappingListDirectoryChangeRowPerPage,
    creditCurveMappingListDirectoryChangePage,
    creditCurveMappingListDirectoryCalculatePagesCount,
    creditCurveMappingListDirectoryTableSort,
    loadCreditCurveMappingDictionary,
    loadCreditCurveMappingData,
    getAccessParams,
  };
  return bindActionCreators(bindObject, dispatch);
}

export default compose(connect(mapStateToProps, mapDispatchToProps), withInitialization)(CreditCurveMappingDirectory);
