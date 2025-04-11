import Button from '@sber-risks-ui/core/button';
import FlexContainer from '@sber-risks-ui/core/flex-container';
import FormControl from '@sber-risks-ui/core/form-control';
import IconButton from '@sber-risks-ui/core/icon-button';
import Typography from '@sber-risks-ui/core/typography';
import React, {Fragment, PureComponent} from 'react';
import {connect} from 'react-redux';
import {bindActionCreators, compose} from 'redux';

import AccessDenied from '../../components/access-denied';
import HeaderRowContainer from '../../components/common/header-row-container';
import Paper from '../../components/common/paper';
import DirectoriesSearchLineContainer from '../../components/directories/directories-search-line-container';
import DirectorySearchLine from '../../components/directories/directory-search-line';
import MassSettlementCalendarComponent from '../../components/directories/mass-settlement-calendar';
import {
  CancelTitle,
  DirectoryTitle,
  MassSettlementCalendarTitle,
  SaveTitle,
} from '../../components/formatted-messages';
import LoadProgress from '../../components/load-progress';
import MainContainer from '../../components/main-container';
import MainNavigation from '../../components/main-navigation';
import getUserAccessDom from '../../helpers/get-user-access-dom';
import isNotEmptyString from '../../helpers/is-not-empty-string';
import massSettlementCalendarEventsCompare from '../../helpers/mass-settlement-calendar-events-compare';
import withInitialization from '../../hocs/with-initialization';
import {getAccessParams} from '../../redux/actions';
import LoadStatus from '../../enums/load-status';
import {
  loadMassSettlementCalendarDirectory,
  loadMassSettlementCalendarDirectoryData,
  massSettlementCalendarActiveIndexColumnChange,
  massSettlementCalendarActiveIndexRowChange,
  massSettlementCalendarChangeActiveCalendar,
  massSettlementCalendarChangeVisibleConfirmDialog,
  massSettlementCalendarEventRemove,
  massSettlementCalendarFilterCalendarChange,
  massSettlementCalendarFilterEventChange,
  massSettlementCalendarInsert,
  massSettlementCalendarInsertNewItem,
  massSettlementCalendarItemChange,
  massSettlementCalendarListDirectoryCalculatePagesCount,
  massSettlementCalendarListDirectoryChangePage,
  massSettlementCalendarListDirectoryChangeRowPerPage,
  massSettlementCalendarListDirectoryTableSort,
  massSettlementCalendarTabsExpanded,
  massSettlementCalendarTabsPanelCalendarExpandedChange,
} from '../../redux/mass-settlement-calendar-directory/actions';
import {
  getActiveCalendarIndex,
  getLoadStatusCalendarInsert,
  getLoadStatusMassSettlementCalendar,
  getMassSettlementCalendarActiveIndexColumnItems,
  getMassSettlementCalendarActiveIndexRowItems,
  getMassSettlementCalendarArrayDeletedItems,
  getMassSettlementCalendarArrayEditedItems,
  getMassSettlementCalendarFiltered,
  getMassSettlementCalendarIsChangeAnyField,
  getMassSettlementCalendarIsTabCalendarExpand,
  getMassSettlementCalendarIsTabExpand,
  getMassSettlementCalendarIsVisibleConfirmDialog,
  getMassSettlementCalendarPaginator,
  getMassSettlementCalendarRowsPerPage,
  getMassSettlementCalendarSearchCalendars,
  getMassSettlementCalendarSearchEvents,
  getMassSettlementCalendarSortByField,
  getMassSettlementCalendarSortDirection,
  getMassSettlementCalendarTabs,
  getRouteName,
} from '../../redux/mass-settlement-calendar-directory/selectors';
import IMassSettlementCalendarEvent
  from '../../redux/mass-settlement-calendar-directory/types/mass-settlement-calendar-event';
import ILoanPricing from '../../redux/types/iloan-pricing';
import IPaginator from '../../types/paginator';
import IMassSettlementCalendarDirectoryComponent from './imass-settlement-calendar-directory';
import Icon from '@sber-risks-ui/icon';

class MassSettlementCalendarDirectory extends PureComponent<IMassSettlementCalendarDirectoryComponent, {}> {
  constructor(props: IMassSettlementCalendarDirectoryComponent) {
    super(props);
    this.onTenantChangeApply = this.onTenantChangeApply.bind(this);
    this.pageChangeHandler = this.pageChangeHandler.bind(this);
    this.rowsPerPageChangeHandler = this.rowsPerPageChangeHandler.bind(this);
    this.calculatePagesCount = this.calculatePagesCount.bind(this);
    this.onInputSearchChange = this.onInputSearchChange.bind(this);
    this.onActiveIndexRowChange = this.onActiveIndexRowChange.bind(this);
    this.onActiveIndexColumnChange = this.onActiveIndexColumnChange.bind(this);
  }

  componentDidMount() {
    this.props.loadMassSettlementCalendarDirectory();
  }

  pageChangeHandler(evt: React.MouseEvent<HTMLButtonElement> | null, page: number) {
    const {props} = this;
    props.massSettlementCalendarListDirectoryChangePage(page);
    this.calculatePagesCount();
    this.props.massSettlementCalendarActiveIndexRowChange(null);
  }

  rowsPerPageChangeHandler(evt: any) {
    this.props.massSettlementCalendarListDirectoryChangeRowPerPage(evt.target.value);
    this.calculatePagesCount();
  }

  calculatePagesCount() {
    const {props} = this;
    const countPage: number = props.massSettlementCalendarItem.events.length;
    props.massSettlementCalendarListDirectoryCalculatePagesCount(countPage);
  }

  onTenantChangeApply() {
    const cb = () => {
      this.props.loadMassSettlementCalendarDirectoryData();
    };
    this.props.getAccessParams(this.props.routeName, cb, true);
  }

  onInputSearchChange(searchValue: string) {
    this.props.massSettlementCalendarFilterEventChange(searchValue);
  }

  onActiveIndexRowChange(index: number) {
    this.props.massSettlementCalendarActiveIndexRowChange(index);
  }

  onActiveIndexColumnChange(index: number) {
    this.props.massSettlementCalendarActiveIndexColumnChange(index);
  }

  massSettlementCalendarDirectoryData() {
    const {props} = this;
    const {page, rowsPerPage}: IPaginator = props.paginator;
    const massSettlementCalendarItemEvents: any[] = props.massSettlementCalendarItem?.events ?? [];
    const massSettlementCalendarCount: number = massSettlementCalendarItemEvents.length || 0;
    const onBtnSaveClick = () => {
      props.massSettlementCalendarInsert(props.massSettlementCalendarItem.storageKey);
    };
    const locale: string = props.locale;

    const onButtonCancelClick = () => {
      props.loadMassSettlementCalendarDirectoryData();
    };

    const eventsWithoutDeleted: IMassSettlementCalendarEvent[] =
      props.massSettlementCalendarItem?.events?.filter(
        (massSettlementCalendarElement: IMassSettlementCalendarEvent) =>
          !props.arrayDeletedItems.find(
            (deletedElement: string) => massSettlementCalendarElement.uniqueId === deletedElement
          )
      ) ?? [];
    const isNotValidDates: boolean =
      eventsWithoutDeleted?.filter(
        (massSettlementCalendarElement: IMassSettlementCalendarEvent) =>
          massSettlementCalendarElement.date === null ||
          massSettlementCalendarElement.date === '' ||
          massSettlementCalendarElement.date === undefined
      )?.length > 0 || !massSettlementCalendarEventsCompare(eventsWithoutDeleted);

    const massSettlementCalendarDirectoryBtnCancelSaveCalendarAccess = getUserAccessDom(
      props.userAccess.massSettlementCalendarDirectoryBtnCancelSaveCalendar.action
    );

    const massSettlementCalendarDirectoryBtnSaveCalendarAccess = getUserAccessDom(
      props.userAccess.massSettlementCalendarDirectoryBtnSaveCalendar.action
    );

    const isDisabledBtnSave: boolean =
      isNotValidDates || massSettlementCalendarDirectoryBtnSaveCalendarAccess.isDisable || !props.isChangeAnyField;

    const isDisabledBtnCancel: boolean =
      massSettlementCalendarDirectoryBtnCancelSaveCalendarAccess.isDisable || !props.isChangeAnyField;

    if (massSettlementCalendarItemEvents && massSettlementCalendarCount <= rowsPerPage * page && page > 0) {
      props.massSettlementCalendarListDirectoryChangePage(0);
    }

    return (
      <Fragment>
        <FormControl minHeight="85vh" justifyContent="space-between" flexDirection="column" width="initial">
          <MassSettlementCalendarComponent
            massSettlementCalendarTabs={props.massSettlementCalendarTabs}
            activeCalendarIndex={props.activeCalendarIndex}
            massSettlementCalendarItem={props.massSettlementCalendarItem}
            sortDirection={props.sortDirection}
            sortByField={props.sortByField}
            paginator={props.paginator}
            massSettlementCalendarListDirectoryTableSort={props.massSettlementCalendarListDirectoryTableSort}
            massSettlementCalendarChangeActiveCalendar={props.massSettlementCalendarChangeActiveCalendar}
            massSettlementCalendarItemChange={props.massSettlementCalendarItemChange}
            arrayEditedItems={props.arrayEditedItems}
            arrayDeletedItems={props.arrayDeletedItems}
            onActiveIndexRowChange={this.onActiveIndexRowChange}
            onActiveIndexColumnChange={this.onActiveIndexColumnChange}
            activeIndexRow={props.activeIndexRow}
            activeIndexColumn={props.activeIndexColumn}
            massSettlementCalendarEventRemove={props.massSettlementCalendarEventRemove}
            locale={locale}
            massSettlementCalendarChangeVisibleConfirmDialog={props.massSettlementCalendarChangeVisibleConfirmDialog}
            isVisibleConfirmDialog={props.isVisibleConfirmDialog}
            loadMassSettlementCalendarDirectoryData={props.loadMassSettlementCalendarDirectoryData}
            isChangeAnyField={props.isChangeAnyField}
            isTabExpanded={props.isTabExpanded}
            massSettlementCalendarTabsExpanded={props.massSettlementCalendarTabsExpanded}
            massSettlementCalendarFilterCalendarChange={props.massSettlementCalendarFilterCalendarChange}
            searchCalendars={props.searchCalendars}
            isTabsPanelCalendarExpand={props.isTabsPanelCalendarExpand}
            massSettlementCalendarTabsPanelCalendarExpandedChange={
              props.massSettlementCalendarTabsPanelCalendarExpandedChange
            }
            userAccessMassSettlementCalendarDirectoryBtnEditEvent={
              props.userAccess.massSettlementCalendarDirectoryBtnEditEvent
            }
            userAccessMassSettlementCalendarDirectoryBtnRemoveEvent={
              props.userAccess.massSettlementCalendarDirectoryBtnRemoveEvent
            }
            pageChangeHandler={this.pageChangeHandler}
            rowsPerPageChangeHandler={this.rowsPerPageChangeHandler}
            rowsPerPage={props.rowsPerPage}
            calendarItemsLength={props.massSettlementCalendarItem?.events?.length}
          />
          <FlexContainer justifyContent="flex-end" flexDirection="column">
            <FormControl justifyContent="flex-end" margin="5px 0 0 0" width="75%">
              {!massSettlementCalendarDirectoryBtnCancelSaveCalendarAccess.isHide && (
                <Button onClick={onButtonCancelClick} disabled={isDisabledBtnCancel}>
                  <CancelTitle/>
                </Button>
              )}
              {!massSettlementCalendarDirectoryBtnSaveCalendarAccess.isHide && (
                <Button onClick={onBtnSaveClick} disabled={isDisabledBtnSave}>
                  <SaveTitle/>
                </Button>
              )}
            </FormControl>
          </FlexContainer>
        </FormControl>
      </Fragment>
    );
  }

  render() {
    const {props} = this;
    const {userAccess, loadStatusAccessData} = props;
    const {isAccessDenied, massSettlementCalendarDirectoryRoute, massSettlementCalendarDirectoryBtnAddEvent} =
      userAccess;
    const isAccessDeniedComponent = massSettlementCalendarDirectoryRoute.action !== 'SHOW';
    const {status}: { status: string } = props.loadStatusMassSettlementCalendar;
    const {status: statusCalendarInsert}: { status: string } = props.loadStatusCalendarInsert;
    const isLoadingStatus: boolean = status === LoadStatus.load;
    const isLoadingStatusCalendarInsert: boolean = statusCalendarInsert === LoadStatus.load;
    const isLoading: boolean = isLoadingStatus || isLoadingStatusCalendarInsert;
    const isInitial: boolean = status === LoadStatus.initial;

    const massSettlementCalendarDirectoryBtnAddEventAccess = getUserAccessDom(
      massSettlementCalendarDirectoryBtnAddEvent.action
    );

    const {status: loadStatusAccess, message: loadStatusMessage}: { message: string, status: string } =
      loadStatusAccessData;
    const isLoadStatusAccessLoaded: boolean = loadStatusAccess === LoadStatus.loaded;
    const isLoadStatusAccessLoad: boolean = loadStatusAccess === LoadStatus.load;
    const isLoadStatusAccessInitial: boolean = loadStatusAccess === LoadStatus.initial;
    const isLoadStatusAccessLoadedFailed: boolean = loadStatusAccess === LoadStatus.loadingFailed;
    return (
      <Fragment>
        <MainNavigation onTenantChangeApply={this.onTenantChangeApply}>
          <HeaderRowContainer id="mass-settlement-calendar-directory">
            <Typography variant="H1">
              <DirectoryTitle/>: <MassSettlementCalendarTitle/>
            </Typography>
            {!isAccessDenied && !isAccessDeniedComponent && (
              <Fragment>
                {!massSettlementCalendarDirectoryBtnAddEventAccess.isHide && (
                  <IconButton
                    id="Mass_settlement_calendar_directory_btn_addEvent"
                    onClick={props.massSettlementCalendarInsertNewItem}
                    disabled={
                      massSettlementCalendarDirectoryBtnAddEventAccess.isDisable ||
                      props.massSettlementCalendarItem?.events?.length === 0 ||
                      isNotEmptyString(props.searchEvents)
                    }
                  >
                    <Icon name="ic-24-plus"/>
                  </IconButton>
                )}
                <DirectoriesSearchLineContainer>
                  <DirectorySearchLine
                    id="mass-settlement-calendar"
                    name="mass-settlement-calendar-directory-search"
                    searchString={props.searchEvents}
                    onSearchStringChange={this.onInputSearchChange}
                  />
                </DirectoriesSearchLineContainer>
              </Fragment>
            )}
          </HeaderRowContainer>
          <MainContainer loadStatusUserData={props.loadStatusMassSettlementCalendar}>
            <Paper>
              {(isLoading || isLoadStatusAccessLoad) && <LoadProgress/>}
              {!isInitial &&
                !isLoading &&
                !isAccessDenied &&
                !isAccessDeniedComponent &&
                this.massSettlementCalendarDirectoryData()}
              {(isLoadStatusAccessLoaded || isLoadStatusAccessLoadedFailed) && isAccessDenied && (
                <AccessDenied message="Система Loan Pricing" comments={loadStatusMessage}/>
              )}
              {!isLoadStatusAccessInitial && isLoadStatusAccessLoaded && isAccessDeniedComponent && (
                <AccessDenied
                  message="Mass settlement calendar directory"
                  comments={massSettlementCalendarDirectoryRoute.message}
                />
              )}
            </Paper>
          </MainContainer>
        </MainNavigation>
      </Fragment>
    );
  }
}

function mapStateToProps(state: ILoanPricing) {
  return {
    loadStatusMassSettlementCalendar: getLoadStatusMassSettlementCalendar(state),
    massSettlementCalendarTabs: getMassSettlementCalendarTabs(state),
    activeCalendarIndex: getActiveCalendarIndex(state),
    massSettlementCalendarItem: getMassSettlementCalendarFiltered(state),
    paginator: getMassSettlementCalendarPaginator(state),
    sortDirection: getMassSettlementCalendarSortDirection(state),
    sortByField: getMassSettlementCalendarSortByField(state),
    rowsPerPage: getMassSettlementCalendarRowsPerPage(state),
    arrayEditedItems: getMassSettlementCalendarArrayEditedItems(state),
    arrayDeletedItems: getMassSettlementCalendarArrayDeletedItems(state),
    loadStatusCalendarInsert: getLoadStatusCalendarInsert(state),
    activeIndexRow: getMassSettlementCalendarActiveIndexRowItems(state),
    activeIndexColumn: getMassSettlementCalendarActiveIndexColumnItems(state),
    isVisibleConfirmDialog: getMassSettlementCalendarIsVisibleConfirmDialog(state),
    isChangeAnyField: getMassSettlementCalendarIsChangeAnyField(state),
    isTabExpanded: getMassSettlementCalendarIsTabExpand(state),
    searchCalendars: getMassSettlementCalendarSearchCalendars(state),
    searchEvents: getMassSettlementCalendarSearchEvents(state),
    isTabsPanelCalendarExpand: getMassSettlementCalendarIsTabCalendarExpand(state),
    routeName: getRouteName(state),
  };
}

function mapDispatchToProps(dispatch: any) {
  const bindObject = {
    loadMassSettlementCalendarDirectory,
    massSettlementCalendarListDirectoryChangePage,
    massSettlementCalendarListDirectoryChangeRowPerPage,
    massSettlementCalendarListDirectoryCalculatePagesCount,
    massSettlementCalendarListDirectoryTableSort,
    massSettlementCalendarChangeActiveCalendar,
    massSettlementCalendarItemChange,
    loadMassSettlementCalendarDirectoryData,
    massSettlementCalendarInsert,
    massSettlementCalendarActiveIndexRowChange,
    massSettlementCalendarActiveIndexColumnChange,
    massSettlementCalendarEventRemove,
    massSettlementCalendarInsertNewItem,
    massSettlementCalendarChangeVisibleConfirmDialog,
    massSettlementCalendarTabsExpanded,
    massSettlementCalendarFilterCalendarChange,
    massSettlementCalendarFilterEventChange,
    massSettlementCalendarTabsPanelCalendarExpandedChange,
    getAccessParams,
  };
  return bindActionCreators(bindObject, dispatch);
}

export default compose(
  connect(mapStateToProps, mapDispatchToProps),
  withInitialization
)(MassSettlementCalendarDirectory);

