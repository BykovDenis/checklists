import FormControl from '@sber-risks-ui/core/form-control';
import IconButton from '@sber-risks-ui/core/icon-button';
import Typography from '@sber-risks-ui/core/typography';
import React, {Fragment} from 'react';
import {connect} from 'react-redux';
import {bindActionCreators, compose} from 'redux';

import AccessDenied from '../../components/access-denied';
import HeaderRowContainer from '../../components/common/header-row-container';
import Paper from '../../components/common/paper';
import DirectorySearchLine from '../../components/directories/directory-search-line';
import FloatingRateCardDirectory from '../../components/directories/floating-rate-card-directory';
import FloatingRateListDirectory from '../../components/directories/floating-rate-list-directory';
import {DirectoryTitle, FloatingRateIndicatorsTitle} from '../../components/formatted-messages';
import LoadProgress from '../../components/load-progress';
import MainContainer from '../../components/main-container';
import MainNavigation from '../../components/main-navigation';
import getUserAccessDom from '../../helpers/get-user-access-dom';
import withFloatingRateSettingsCardDirectory
  from '../../hocs/directories/floating-rate-settings/with-floating-rate-settings-card-directory';
import withFloatingRateSettingsListDirectory
  from '../../hocs/directories/floating-rate-settings/with-floating-rate-settings-list-directory';
import withInitialization from '../../hocs/with-initialization';
import {getAccessParams} from '../../redux/actions';
import LoadStatus from '../../enums/load-status';
import {
  floatingRateSettingsChangeVisibleCard
} from '../../redux/floating-rate-settings/actions/floating-rate-settings-card-directory';
import {
  floatingRateSettingsDocumentCreate,
  floatingRateSettingsSearchStringChange,
  loadFloatingRateSettings,
  loadFloatingRateSettingsData,
} from '../../redux/floating-rate-settings/actions/floating-rate-settings-list-directory';
import {
  getFloatingRateSettingsSearchString,
  getLoadStatusFloatingRateDirectory,
  getRouteName,
} from '../../redux/floating-rate-settings/selector';
import {getUserAccess} from '../../redux/selectors';
import ILoanPricing from '../../redux/types/iloan-pricing';
import IUserAccess from '../../types/user-access';
import IFloatingRateSettings from './ifloating-rate-settings';
import Icon from '@sber-risks-ui/icon';
import FlexContainer from "@sber-risks-ui/core/flex-container";
import PageWidth from '../../enums/page-width';

const FloatingRateCardDirectoryWrapped: React.FunctionComponent<any> =
  withFloatingRateSettingsCardDirectory(FloatingRateCardDirectory);

const FloatingRateListDirectoryWrapped: React.FunctionComponent<any> =
  withFloatingRateSettingsListDirectory(FloatingRateListDirectory);

class FloatingRateSettings extends React.PureComponent<IFloatingRateSettings, {}> {
  constructor(props: any) {
    super(props);
    this.onTenantChangeApply = this.onTenantChangeApply.bind(this);
    this.onBtnCreateItemsClick = this.onBtnCreateItemsClick.bind(this);
    this.onSearchStringChange = this.onSearchStringChange.bind(this);
    this.onSearchStringClean = this.onSearchStringClean.bind(this);
  }

  componentDidMount() {a
    this.props.loadFloatingRateSettings();
  }

  onTenantChangeApply() {
    const cb = () => {
      this.props.loadFloatingRateSettingsData();
    };
    this.props.getAccessParams(this.props.routeName, cb, true);
  }

  onBtnCreateItemsClick() {
    this.props.floatingRateSettingsChangeVisibleCard();
    this.props.floatingRateSettingsDocumentCreate();
  }

  onSearchStringChange(searchValue: string) {
    this.props.floatingRateSettingsSearchStringChange(searchValue);
  }

  onSearchStringClean() {
    this.props.floatingRateSettingsSearchStringChange('');
  }

  floatingRateSettingsData() {
    return (
      <FormControl flexDirection="column" alignItems="flex-start" margin="30px 0 0 0">
        <FloatingRateCardDirectoryWrapped/>
        <FloatingRateListDirectoryWrapped/>
      </FormControl>
    );
  }

  render() {
    const {props} = this;
    const {userAccess, loadStatusAccessData} = props;
    const {
      isAccessDenied,
      floatingRateSettingsDirectoryRoute,
    }: { floatingRateSettingsDirectoryRoute: IUserAccess, isAccessDenied: boolean } = userAccess;
    const isAccessDeniedComponent: boolean = floatingRateSettingsDirectoryRoute.action !== 'SHOW';
    const {status}: { status: string } = props.loadStatusFloatingRateDirectory;
    const isLoading = status === LoadStatus.load;
    const isInitial = status === LoadStatus.initial;
    const {status: loadStatusAccess, message: loadStatusMessage}: { message: string, status: string } =
      loadStatusAccessData;
    const isLoadStatusAccessLoaded: boolean = loadStatusAccess === LoadStatus.loaded;
    const isLoadStatusAccessLoad: boolean = loadStatusAccess === LoadStatus.load;
    const isLoadStatusAccessInitial: boolean = loadStatusAccess === LoadStatus.initial;
    const isLoadStatusAccessLoadedFailed: boolean = loadStatusAccess === LoadStatus.loadingFailed;

    const floatingRateSettingsDirectoryBtnCreateAction: string =
      props.floatingRateSettingsDirectoryBtnCreate?.action ?? null;
    const floatingRateSettingsDirectoryBtnCreateAccess: any = getUserAccessDom(
      floatingRateSettingsDirectoryBtnCreateAction
    );

    return (
      <Fragment>
        <MainNavigation onTenantChangeApply={this.onTenantChangeApply}>
          <HeaderRowContainer id="counterparty-directory" width={PageWidth.floatingRateSettingsDirectory}>
            <FlexContainer left={15} top={0} position="sticky" width={1206}>
              <Typography variant="H1">
                <DirectoryTitle/>: <FloatingRateIndicatorsTitle/> &nbsp;&nbsp;
              </Typography>
              {!isLoading && !isAccessDenied && !isAccessDeniedComponent && (
                <Fragment>
                  {floatingRateSettingsDirectoryBtnCreateAccess.isAccess && (
                    <IconButton
                      id="Floating_rate_settings_directory_btn_create"
                      onClick={this.onBtnCreateItemsClick}
                      disabled={floatingRateSettingsDirectoryBtnCreateAccess.isDisable}
                    >
                      <Icon name="ic-24-plus"/>
                    </IconButton>
                  )}
                  <DirectorySearchLine
                    id="floating-rate-settings-directory-search"
                    name="floating-rate-settings-directory-search"
                    searchString={props.searchString}
                    onSearchStringChange={this.onSearchStringChange}
                    width={600}
                  />
                </Fragment>
              )}
            </FlexContainer>
          </HeaderRowContainer>
          <MainContainer loadStatusUserData={props.loadStatusUserData}>
            <Paper width={PageWidth.floatingRateSettingsDirectory}>
              {(isLoading || isLoadStatusAccessLoad) && <LoadProgress/>}
              {!isInitial &&
                !isLoading &&
                !isAccessDenied &&
                !isAccessDeniedComponent &&
                this.floatingRateSettingsData()}

              {(isLoadStatusAccessLoaded || isLoadStatusAccessLoadedFailed) && isAccessDenied && (
                <AccessDenied message="Система Loan Pricing" comments={loadStatusMessage}/>
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
  }
}

function mapStateToProps(state: ILoanPricing) {
  return {
    loadStatusFloatingRateDirectory: getLoadStatusFloatingRateDirectory(state),
    searchString: getFloatingRateSettingsSearchString(state),
    floatingRateSettingsDirectoryBtnCreate: getUserAccess(state).floatingRateSettingsDirectoryBtnCreate,
    routeName: getRouteName(state),
  };
}

function mapDispatchToProps(dispatch: any) {
  const bindObject = {
    loadFloatingRateSettings,
    floatingRateSettingsChangeVisibleCard,
    floatingRateSettingsDocumentCreate,
    floatingRateSettingsSearchStringChange,
    loadFloatingRateSettingsData,
    getAccessParams,
  };

  return bindActionCreators(bindObject, dispatch);
}

export default compose(connect(mapStateToProps, mapDispatchToProps), withInitialization)(FloatingRateSettings);
