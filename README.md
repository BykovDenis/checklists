import React, { useEffect, Fragment, useCallback } from 'react';
import { connect } from 'react-redux';
import { bindActionCreators, compose } from 'redux';
import {
  loadFloatingRateSettings,
  floatingRateSettingsChangeVisibleCard,
  floatingRateSettingsDocumentCreate,
  floatingRateSettingsSearchStringChange,
  loadFloatingRateSettingsData,
  getAccessParams,
} from '../../redux/floating-rate-settings/actions/floating-rate-settings-list-directory';
import {
  getFloatingRateSettingsSearchString,
  getLoadStatusFloatingRateDirectory,
  getRouteName,
} from '../../redux/floating-rate-settings/selector';
import { getUserAccess } from '../../redux/selectors';
import ILoanPricing from '../../redux/types/iloan-pricing';
import IUserAccess from '../../types/user-access';
import IFloatingRateSettings from './ifloating-rate-settings';
import LoadStatus from '../../enums/load-status';

import AccessDenied from '../../components/access-denied';
import HeaderRowContainer from '../../components/common/header-row-container';
import Paper from '../../components/common/paper';
import DirectorySearchLine from '../../components/directories/directory-search-line';
import FloatingRateCardDirectory from '../../components/directories/floating-rate-card-directory';
import FloatingRateListDirectory from '../../components/directories/floating-rate-list-directory';
import { DirectoryTitle, FloatingRateIndicatorsTitle } from '../../components/formatted-messages';
import LoadProgress from '../../components/load-progress';
import MainContainer from '../../components/main-container';
import MainNavigation from '../../components/main-navigation';
import getUserAccessDom from '../../helpers/get-user-access-dom';
import withFloatingRateSettingsCardDirectory from '../../hocs/directories/floating-rate-settings/with-floating-rate-settings-card-directory';
import withFloatingRateSettingsListDirectory from '../../hocs/directories/floating-rate-settings/with-floating-rate-settings-list-directory';
import withInitialization from '../../hocs/with-initialization';

import FormControl from '@sber-risks-ui/core/form-control';
import IconButton from '@sber-risks-ui/core/icon-button';
import Typography from '@sber-risks-ui/core/typography';
import Icon from '@sber-risks-ui/icon';
import FlexContainer from "@sber-risks-ui/core/flex-container";
import PageWidth from '../../enums/page-width';

const FloatingRateCardDirectoryWrapped = withFloatingRateSettingsCardDirectory(FloatingRateCardDirectory);
const FloatingRateListDirectoryWrapped = withFloatingRateSettingsListDirectory(FloatingRateListDirectory);

const FloatingRateSettings: React.FC<IFloatingRateSettings> = (props) => {
  const {
    loadFloatingRateSettings,
    loadFloatingRateSettingsData,
    getAccessParams,
    floatingRateSettingsChangeVisibleCard,
    floatingRateSettingsDocumentCreate,
    floatingRateSettingsSearchStringChange,
    loadStatusFloatingRateDirectory,
    searchString,
    routeName,
    userAccess,
    loadStatusAccessData,
    floatingRateSettingsDirectoryBtnCreate
  } = props;

  useEffect(() => {
    loadFloatingRateSettings();
  }, [loadFloatingRateSettings]);

  const onTenantChangeApply = useCallback(() => {
    const cb = () => loadFloatingRateSettingsData();
    getAccessParams(routeName, cb, true);
  }, [getAccessParams, routeName, loadFloatingRateSettingsData]);

  const onBtnCreateItemsClick = () => {
    floatingRateSettingsChangeVisibleCard();
    floatingRateSettingsDocumentCreate();
  };

  const onSearchStringChange = (value: string) => {
    floatingRateSettingsSearchStringChange(value);
  };

  const onSearchStringClean = () => {
    floatingRateSettingsSearchStringChange('');
  };

  const {
    isAccessDenied,
    floatingRateSettingsDirectoryRoute,
  }: { floatingRateSettingsDirectoryRoute: IUserAccess, isAccessDenied: boolean } = userAccess;

  const isAccessDeniedComponent = floatingRateSettingsDirectoryRoute.action !== 'SHOW';
  const status = loadStatusFloatingRateDirectory.status;
  const isLoading = status === LoadStatus.load;
  const isInitial = status === LoadStatus.initial;

  const { status: loadStatusAccess, message: loadStatusMessage } = loadStatusAccessData;
  const isLoadStatusAccessLoaded = loadStatusAccess === LoadStatus.loaded;
  const isLoadStatusAccessLoad = loadStatusAccess === LoadStatus.load;
  const isLoadStatusAccessInitial = loadStatusAccess === LoadStatus.initial;
  const isLoadStatusAccessLoadedFailed = loadStatusAccess === LoadStatus.loadingFailed;

  const floatingRateSettingsDirectoryBtnCreateAction = floatingRateSettingsDirectoryBtnCreate?.action ?? null;
  const floatingRateSettingsDirectoryBtnCreateAccess = getUserAccessDom(floatingRateSettingsDirectoryBtnCreateAction);

  return (
    <Fragment>
      <MainNavigation onTenantChangeApply={onTenantChangeApply}>
        <HeaderRowContainer id="counterparty-directory" width={PageWidth.floatingRateSettingsDirectory}>
          <FlexContainer left={15} top={0} position="sticky" width={1206}>
            <Typography variant="H1">
              <DirectoryTitle />: <FloatingRateIndicatorsTitle /> &nbsp;&nbsp;
            </Typography>
            {!isLoading && !isAccessDenied && !isAccessDeniedComponent && (
              <Fragment>
                {floatingRateSettingsDirectoryBtnCreateAccess.isAccess && (
                  <IconButton
                    id="Floating_rate_settings_directory_btn_create"
                    onClick={onBtnCreateItemsClick}
                    disabled={floatingRateSettingsDirectoryBtnCreateAccess.isDisable}
                  >
                    <Icon name="ic-24-plus" />
                  </IconButton>
                )}
                <DirectorySearchLine
                  id="floating-rate-settings-directory-search"
                  name="floating-rate-settings-directory-search"
                  searchString={searchString}
                  onSearchStringChange={onSearchStringChange}
                  width={600}
                />
              </Fragment>
            )}
          </FlexContainer>
        </HeaderRowContainer>

        <MainContainer loadStatusUserData={props.loadStatusUserData}>
          <Paper width={PageWidth.floatingRateSettingsDirectory}>
            {(isLoading || isLoadStatusAccessLoad) && <LoadProgress />}
            {!isInitial && !isLoading && !isAccessDenied && !isAccessDeniedComponent && (
              <>
                <FloatingRateCardDirectoryWrapped />
                <FloatingRateListDirectoryWrapped />
              </>
            )}
            {(isLoadStatusAccessLoaded || isLoadStatusAccessLoadedFailed) && isAccessDenied && (
              <AccessDenied message="Система Loan Pricing" comments={loadStatusMessage} />
            )}
            {!isLoadStatusAccessInitial && isLoadStatusAccessLoaded && isAccessDeniedComponent && (
              <AccessDenied
                message="Floating rate settings directory"
                comments={floatingRateSettingsDirectoryRoute.message}
              />
            )}
          </Paper>
        </MainContainer>
      </MainNavigation>
    </Fragment>
  );
};

function mapStateToProps(state: ILoanPricing) {
  return {
    loadStatusFloatingRateDirectory: getLoadStatusFloatingRateDirectory(state),
    searchString: getFloatingRateSettingsSearchString(state),
    floatingRateSettingsDirectoryBtnCreate: getUserAccess(state).floatingRateSettingsDirectoryBtnCreate,
    routeName: getRouteName(state),
    userAccess: getUserAccess(state),
    loadStatusAccessData: state.loadStatusAccessData,
    loadStatusUserData: state.loadStatusUserData,
  };
}

function mapDispatchToProps(dispatch: any) {
  return bindActionCreators({
    loadFloatingRateSettings,
    floatingRateSettingsChangeVisibleCard,
    floatingRateSettingsDocumentCreate,
    floatingRateSettingsSearchStringChange,
    loadFloatingRateSettingsData,
    getAccessParams,
  }, dispatch);
}

export default compose(
  connect(mapStateToProps, mapDispatchToProps),
  withInitialization
)(FloatingRateSettings);
