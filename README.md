import React, { useCallback, useEffect, Fragment } from 'react';
import { connect } from 'react-redux';
import { bindActionCreators, compose } from 'redux';

import Button from '@sber-risks-ui/core/button';
import FlexContainer from '@sber-risks-ui/core/flex-container';
import FormControl from '@sber-risks-ui/core/form-control';
import IconButton from '@sber-risks-ui/core/icon-button';
import Typography from '@sber-risks-ui/core/typography';
import Icon from '@sber-risks-ui/icon';

import AccessDenied from '../../components/access-denied';
import HeaderRowContainer from '../../components/common/header-row-container';
import Paper from '../../components/common/paper';
import DirectoriesSearchLineContainer from '../../components/directories/directories-search-line-container';
import DirectorySearchLine from '../../components/directories/directory-search-line';
import MassSettlementCalendarComponent from '../../components/directories/mass-settlement-calendar';
import { CancelTitle, DirectoryTitle, MassSettlementCalendarTitle, SaveTitle } from '../../components/formatted-messages';
import LoadProgress from '../../components/load-progress';
import MainContainer from '../../components/main-container';
import MainNavigation from '../../components/main-navigation';
import getUserAccessDom from '../../helpers/get-user-access-dom';
import isNotEmptyString from '../../helpers/is-not-empty-string';
import massSettlementCalendarEventsCompare from '../../helpers/mass-settlement-calendar-events-compare';
import withInitialization from '../../hocs/with-initialization';

import * as actions from '../../redux/mass-settlement-calendar-directory/actions';
import * as selectors from '../../redux/mass-settlement-calendar-directory/selectors';
import { getAccessParams } from '../../redux/actions';
import LoadStatus from '../../enums/load-status';

const MassSettlementCalendarDirectory = (props) => {
  const {
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
    ...restProps
  } = props;

  useEffect(() => {
    loadMassSettlementCalendarDirectory();
  }, [loadMassSettlementCalendarDirectory]);

  const pageChangeHandler = useCallback((evt, page) => {
    massSettlementCalendarListDirectoryChangePage(page);
    calculatePagesCount();
    massSettlementCalendarActiveIndexRowChange(null);
  }, [massSettlementCalendarListDirectoryChangePage, calculatePagesCount, massSettlementCalendarActiveIndexRowChange]);

  const rowsPerPageChangeHandler = useCallback((evt) => {
    massSettlementCalendarListDirectoryChangeRowPerPage(evt.target.value);
    calculatePagesCount();
  }, [massSettlementCalendarListDirectoryChangeRowPerPage, calculatePagesCount]);

  const calculatePagesCount = useCallback(() => {
    const countPage = props.massSettlementCalendarItem.events.length;
    massSettlementCalendarListDirectoryCalculatePagesCount(countPage);
  }, [props.massSettlementCalendarItem, massSettlementCalendarListDirectoryCalculatePagesCount]);

  const onTenantChangeApply = useCallback(() => {
    const cb = () => {
      loadMassSettlementCalendarDirectoryData();
    };
    getAccessParams(props.routeName, cb, true);
  }, [getAccessParams, loadMassSettlementCalendarDirectoryData, props.routeName]);

  const onInputSearchChange = useCallback((searchValue) => {
    massSettlementCalendarFilterEventChange(searchValue);
  }, [massSettlementCalendarFilterEventChange]);

  const onActiveIndexRowChange = useCallback((index) => {
    massSettlementCalendarActiveIndexRowChange(index);
  }, [massSettlementCalendarActiveIndexRowChange]);

  const onActiveIndexColumnChange = useCallback((index) => {
    massSettlementCalendarActiveIndexColumnChange(index);
  }, [massSettlementCalendarActiveIndexColumnChange]);

  const renderCalendarDirectoryData = () => {
    const {
      paginator, massSettlementCalendarItem, arrayDeletedItems, isChangeAnyField, locale,
      userAccess, sortByField, sortDirection, arrayEditedItems, isVisibleConfirmDialog,
      isTabExpanded, searchCalendars, isTabsPanelCalendarExpand, rowsPerPage, activeIndexRow, activeIndexColumn,
    } = props;

    const { page } = paginator;
    const events = massSettlementCalendarItem?.events ?? [];
    const filteredEvents = events.filter(e => !arrayDeletedItems.includes(e.uniqueId));
    const isInvalidDates = filteredEvents.some(e => !e.date) || !massSettlementCalendarEventsCompare(filteredEvents);

    const btnCancelAccess = getUserAccessDom(userAccess.massSettlementCalendarDirectoryBtnCancelSaveCalendar.action);
    const btnSaveAccess = getUserAccessDom(userAccess.massSettlementCalendarDirectoryBtnSaveCalendar.action);

    const isSaveDisabled = isInvalidDates || btnSaveAccess.isDisable || !isChangeAnyField;
    const isCancelDisabled = btnCancelAccess.isDisable || !isChangeAnyField;

    if (events.length && events.length <= rowsPerPage * page && page > 0) {
      massSettlementCalendarListDirectoryChangePage(0);
    }

    return (
      <FormControl minHeight="85vh" justifyContent="space-between" flexDirection="column" width="initial">
        <MassSettlementCalendarComponent
          {...{
            ...props,
            onActiveIndexRowChange,
            onActiveIndexColumnChange,
            pageChangeHandler,
            rowsPerPageChangeHandler,
          }}
        />
        <FlexContainer justifyContent="flex-end" flexDirection="column">
          <FormControl justifyContent="flex-end" margin="5px 0 0 0" width="75%">
            {!btnCancelAccess.isHide && (
              <Button onClick={loadMassSettlementCalendarDirectoryData} disabled={isCancelDisabled}>
                <CancelTitle />
              </Button>
            )}
            {!btnSaveAccess.isHide && (
              <Button onClick={() => massSettlementCalendarInsert(massSettlementCalendarItem.storageKey)} disabled={isSaveDisabled}>
                <SaveTitle />
              </Button>
            )}
          </FormControl>
        </FlexContainer>
      </FormControl>
    );
  };

  const {
    userAccess,
    loadStatusMassSettlementCalendar,
    loadStatusCalendarInsert,
    loadStatusAccessData,
    massSettlementCalendarDirectoryRoute,
    massSettlementCalendarDirectoryBtnAddEvent,
    searchEvents,
  } = props;

  const isAccessDenied = userAccess.isAccessDenied || massSettlementCalendarDirectoryRoute.action !== 'SHOW';
  const isLoading = [loadStatusMassSettlementCalendar.status, loadStatusCalendarInsert.status].includes(LoadStatus.load);
  const isInitial = loadStatusMassSettlementCalendar.status === LoadStatus.initial;

  const addEventAccess = getUserAccessDom(massSettlementCalendarDirectoryBtnAddEvent.action);
  const { status: accessStatus, message: accessMessage } = loadStatusAccessData;

  return (
    <MainNavigation onTenantChangeApply={onTenantChangeApply}>
      <HeaderRowContainer id="mass-settlement-calendar-directory">
        <Typography variant="H1">
          <DirectoryTitle />: <MassSettlementCalendarTitle />
        </Typography>
        {!isAccessDenied && (
          <Fragment>
            {!addEventAccess.isHide && (
              <IconButton
                id="Mass_settlement_calendar_directory_btn_addEvent"
                onClick={massSettlementCalendarInsertNewItem}
                disabled={addEventAccess.isDisable || props.massSettlementCalendarItem?.events?.length === 0 || isNotEmptyString(searchEvents)}
              >
                <Icon name="ic-24-plus" />
              </IconButton>
            )}
            <DirectoriesSearchLineContainer>
              <DirectorySearchLine
                id="mass-settlement-calendar"
                name="mass-settlement-calendar-directory-search"
                searchString={searchEvents}
                onSearchStringChange={onInputSearchChange}
              />
            </DirectoriesSearchLineContainer>
          </Fragment>
        )}
      </HeaderRowContainer>
      <MainContainer loadStatusUserData={loadStatusMassSettlementCalendar}>
        <Paper>
          {(isLoading || accessStatus === LoadStatus.load) && <LoadProgress />}
          {!isInitial && !isLoading && !isAccessDenied && renderCalendarDirectoryData()}
          {accessStatus === LoadStatus.loaded && userAccess.isAccessDenied && (
            <AccessDenied message="Система Loan Pricing" comments={accessMessage} />
          )}
          {accessStatus === LoadStatus.loaded && massSettlementCalendarDirectoryRoute.action !== 'SHOW' && (
            <AccessDenied message="Mass settlement calendar directory" comments={massSettlementCalendarDirectoryRoute.message} />
          )}
        </Paper>
      </MainContainer>
    </MainNavigation>
  );
};

const mapStateToProps = (state) => ({
  loadStatusMassSettlementCalendar: selectors.getLoadStatusMassSettlementCalendar(state),
  massSettlementCalendarTabs: selectors.getMassSettlementCalendarTabs(state),
  activeCalendarIndex: selectors.getActiveCalendarIndex(state),
  massSettlementCalendarItem: selectors.getMassSettlementCalendarFiltered(state),
  paginator: selectors.getMassSettlementCalendarPaginator(state),
  sortDirection: selectors.getMassSettlementCalendarSortDirection(state),
  sortByField: selectors.getMassSettlementCalendarSortByField(state),
  rowsPerPage: selectors.getMassSettlementCalendarRowsPerPage(state),
  arrayEditedItems: selectors.getMassSettlementCalendarArrayEditedItems(state),
  arrayDeletedItems: selectors.getMassSettlementCalendarArrayDeletedItems(state),
  loadStatusCalendarInsert: selectors.getLoadStatusCalendarInsert(state),
  activeIndexRow: selectors.getMassSettlementCalendarActiveIndexRowItems(state),
  activeIndexColumn: selectors.getMassSettlementCalendarActiveIndexColumnItems(state),
  isVisibleConfirmDialog: selectors.getMassSettlementCalendarIsVisibleConfirmDialog(state),
  isChangeAnyField: selectors.getMassSettlementCalendarIsChangeAnyField(state),
  isTabExpanded: selectors.getMassSettlementCalendarIsTabExpand(state),
  searchCalendars: selectors.getMassSettlementCalendarSearchCalendars(state),
  searchEvents: selectors.getMassSettlementCalendarSearchEvents(state),
  isTabsPanelCalendarExpand: selectors.getMassSettlementCalendarIsTabCalendarExpand(state),
  routeName: selectors.getRouteName(state),
});

const mapDispatchToProps = (dispatch) => bindActionCreators({
  ...actions,
  getAccessParams,
}, dispatch);

export default compose(
  connect(mapStateToProps, mapDispatchToProps),
  withInitialization
)(MassSettlementCalendarDirectory);
